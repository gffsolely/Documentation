在 Unity 中可以使用 Firebase Authentication 完成 Google、Facebook、Twitter、Apple 等社交媒体账户的登录。Firebase 提供了适用于 Unity 的 SDK，使得在 Unity 项目中集成这些社交媒体登录变得相对简单。

### 准备工作

1. **创建 Firebase 项目**
   - 登录 [Firebase 控制台](https://console.firebase.google.com/)，创建一个新的 Firebase 项目。

2. **启用所需的身份验证方法**
   - 在 Firebase 控制台中，导航到 "Authentication" 部分，然后在 "Sign-in method" 选项卡中启用你希望使用的身份验证方法（如 Google、Facebook、Twitter、Apple）。

3. **配置 OAuth 供应商**
   - 配置相应的 OAuth 供应商，如 Google、Facebook、Twitter、Apple，并获取相应的客户端 ID 和密钥。

4. **下载 Firebase Unity SDK**
   - 从 [Firebase Unity SDK](https://firebase.google.com/docs/unity/setup) 页面下载并导入 SDK 到你的 Unity 项目中。

### 在 Unity 中集成 Firebase Authentication

#### 1. 设置 Firebase Unity SDK

- 导入 Firebase SDK 包到你的 Unity 项目中。
- 将 `google-services.json`（Android）和 `GoogleService-Info.plist`（iOS）文件添加到你的 Unity 项目中。

#### 2. 初始化 Firebase

在你的 Unity 项目中的一个脚本中初始化 Firebase：

```csharp
using Firebase;
using Firebase.Auth;
using UnityEngine;

public class FirebaseInit : MonoBehaviour
{
    private FirebaseAuth auth;

    void Start()
    {
        FirebaseApp.CheckAndFixDependenciesAsync().ContinueWith(task =>
        {
            FirebaseApp app = FirebaseApp.DefaultInstance;
            auth = FirebaseAuth.DefaultInstance;
        });
    }
}
```

#### 3. 实现 Google 登录

需要在 Google 开发者控制台中配置 Google 登录，然后在 Unity 中实现 Google 登录功能。使用 Firebase SDK for Unity 提供的 `GoogleSignIn` 插件可以简化这一过程。

##### 配置 Google 登录

- 在 Google 开发者控制台中创建 OAuth 2.0 客户端 ID。
- 在 Firebase 控制台中启用 Google 登录并配置 OAuth 客户端 ID。

##### 使用 `GoogleSignIn` 插件

1. **导入 GoogleSignIn 插件**

   从 [Google Sign-In Unity 插件](https://github.com/googlesamples/google-signin-unity) 页面下载并导入到 Unity 项目中。

2. **实现 Google 登录**

```csharp
using Firebase;
using Firebase.Auth;
using Google;
using UnityEngine;

public class GoogleSignInHandler : MonoBehaviour
{
    private FirebaseAuth auth;
    private GoogleSignInConfiguration configuration;

    void Start()
    {
        configuration = new GoogleSignInConfiguration
        {
            WebClientId = "YOUR_WEB_CLIENT_ID",
            RequestEmail = true,
            RequestIdToken = true
        };

        GoogleSignIn.Configuration = configuration;
        auth = FirebaseAuth.DefaultInstance;
    }

    public void SignInWithGoogle()
    {
        GoogleSignIn.DefaultInstance.SignIn().ContinueWith(task =>
        {
            if (task.IsCanceled || task.IsFaulted)
            {
                Debug.LogError("Google sign-in failed.");
                return;
            }

            GoogleSignInUser googleUser = task.Result;
            Credential credential = GoogleAuthProvider.GetCredential(googleUser.IdToken, null);

            auth.SignInWithCredentialAsync(credential).ContinueWith(authTask =>
            {
                if (authTask.IsCanceled || authTask.IsFaulted)
                {
                    Debug.LogError("Firebase sign-in with Google failed.");
                    return;
                }

                FirebaseUser newUser = authTask.Result;
                Debug.LogFormat("User signed in successfully: {0} ({1})", newUser.DisplayName, newUser.UserId);
            });
        });
    }
}
```

#### 4. 实现 Facebook 登录

配置 Facebook 登录类似于 Google 登录，需要在 Facebook 开发者控制台中创建应用并获取 App ID 和 App Secret，然后在 Firebase 控制台中启用 Facebook 登录并配置 OAuth 供应商。

##### 使用 Facebook SDK

1. **导入 Facebook SDK**

   从 [Facebook SDK for Unity](https://developers.facebook.com/docs/unity/) 页面下载并导入到 Unity 项目中。

2. **实现 Facebook 登录**

```csharp
using Facebook.Unity;
using Firebase;
using Firebase.Auth;
using UnityEngine;

public class FacebookSignInHandler : MonoBehaviour
{
    private FirebaseAuth auth;

    void Start()
    {
        if (!FB.IsInitialized)
        {
            FB.Init(() =>
            {
                if (FB.IsInitialized)
                    FB.ActivateApp();
                else
                    Debug.LogError("Failed to initialize Facebook SDK");
            }, isGameShown => { });
        }
        else
        {
            FB.ActivateApp();
        }

        auth = FirebaseAuth.DefaultInstance;
    }

    public void SignInWithFacebook()
    {
        FB.LogInWithReadPermissions(new List<string> { "public_profile", "email" }, result =>
        {
            if (FB.IsLoggedIn)
            {
                AccessToken token = AccessToken.CurrentAccessToken;
                Credential credential = FacebookAuthProvider.GetCredential(token.TokenString);

                auth.SignInWithCredentialAsync(credential).ContinueWith(authTask =>
                {
                    if (authTask.IsCanceled || authTask.IsFaulted)
                    {
                        Debug.LogError("Firebase sign-in with Facebook failed.");
                        return;
                    }

                    FirebaseUser newUser = authTask.Result;
                    Debug.LogFormat("User signed in successfully: {0} ({1})", newUser.DisplayName, newUser.UserId);
                });
            }
            else
            {
                Debug.LogError("Facebook sign-in failed.");
            }
        });
    }
}
```

### 小结

通过 Firebase Authentication 和相应的社交登录 SDK，你可以在 Unity 中轻松实现 Google、Facebook、Twitter、Apple 等社交媒体账户的登录。这需要一些配置工作，包括在 Firebase 控制台中启用相应的登录方法和在相应的社交媒体平台上配置 OAuth 客户端。然后，通过使用 Firebase SDK for Unity 和相应的社交媒体登录插件，你可以在你的 Unity 项目中实现这些登录功能。
