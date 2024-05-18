---
title: "[Unity] SendWebRequest Async await 사용하기"
categories:
  - Programming
  - Unity
tags:
  - community
  - CSharp
media_subpath: /assets/posts/
image:
---
해당 코드를 프로젝트 추가하면
`UnityWebRequestAsyncOperation` 를 Async 함수에서 await로 기다릴 수 있습니다

``` C#
public struct UnityWebRequestAwaiter : INotifyCompletion  
{  
    private UnityWebRequestAsyncOperation asyncOp;  
    private Action continuation;  
  
    public UnityWebRequestAwaiter(UnityWebRequestAsyncOperation asyncOp)  
    {        this.asyncOp = asyncOp;  
        continuation = null;  
    }  
    public bool IsCompleted { get { return asyncOp.isDone; } }  
  
    public void GetResult() { }  
  
    public void OnCompleted(Action continuation)  
    {        this.continuation = continuation;  
        asyncOp.completed += OnRequestCompleted;  
    }  
    private void OnRequestCompleted(AsyncOperation obj)  
    {        continuation?.Invoke();  
    }}  
  
public static class ExtensionMethods  
{  
    public static UnityWebRequestAwaiter GetAwaiter(this UnityWebRequestAsyncOperation asyncOp)  
    {        return new UnityWebRequestAwaiter(asyncOp);  
    }}
```

사용 예시

```C#
var www = UnityWebRequest.Get(url);  
var ao = www.SendWebRequest(); // 응답이 올때까지 대기한다.  
await ao;
```
