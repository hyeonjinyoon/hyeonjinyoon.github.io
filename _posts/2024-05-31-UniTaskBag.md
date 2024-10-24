---
title: "[Unity] UniTaskBag"
categories:
  - Programming
  - Unity
tags:
  - CSharp
  - Unity
media_subpath: /assets/posts/
image:
---

```c#
public class UniTaskBag  
{  
    private readonly Dictionary<CancellationToken, CancellationTokenSource> _tokenSources = new();  
  
    ~UniTaskBag()  
    {        
	    Dispose();  
    }    
    
    public CancellationToken CreateCancelToken()  
    {        var source = new CancellationTokenSource();  
        _tokenSources.Add(source.Token, source);  
        return source.Token;  
    }  
    
    public void Dispose()  
    {        _tokenSources.Foreach(pair =>  
        {  
            var source = pair.Value;  
            source.Cancel();  
            source.Dispose();  
        });    
    }
      
    public void Dispose(CancellationToken token)  
    {        var source = _tokenSources[token];  
        source.Cancel();  
        source.Dispose();  
        _tokenSources.Remove(token);  
    }  
}
```


사용 예시

```c#
public class Test : Monobehaviour  
{  
	private UniTaskBag _uniTaskBag = new();  
  
	private void OnEnable()  
	{  
	    FireAsync(_uniTaskBag.CreateCancelToken()).Forget();  
	    TestAsync(_uniTaskBag.CreateCancelToken()).Forget();  
	      
	    var token = _uniTaskBag.CreateCancelToken();  
	    Test2Async(token).Forget();  
	    _uniTaskBag.Dispose(token);  
	}    
	    
	private void OnDisable()    
	{    
	    _uniTaskBag.Dispose();  
	}  
	...  
}  
```
