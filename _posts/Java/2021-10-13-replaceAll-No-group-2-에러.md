replaceAll 메서드에 $ 사용시 발생한다.

에러는 다음과 같다.
```java
Exception in thread "main" java.lang.IndexOutOfBoundsException: No group 2
	at java.util.regex.Matcher.start(Matcher.java:374)
	at java.util.regex.Matcher.appendReplacement(Matcher.java:831)
	at java.util.regex.Matcher.replaceAll(Matcher.java:906)
	at java.lang.String.replaceAll(String.java:2162)

```

오류는 달러 기호가 포함되어 있어 발생한다.

***Note that backslashes (\) and dollar signs ($) in the replacement string may cause the results to be different than if it were being treated as a literal replacement string; see Matcher.replaceAll. Use Matcher.quoteReplacement(java.lang.String) to suppress the special meaning of these characters, if desired.***

```java
    content = content.replaceAll(
            Regexp.MESSAGE_PREFIX, quoteReplacement(arr[i]));
}
```

quoteReplacement 사용하여 처리하게 되면 에러가 발생하지 않는다.
