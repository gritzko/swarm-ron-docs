## Compression

Compression is a format-neutral trick.
Every spec has exactly four components.
As subsequent op specs are more likely to share some tokens, such repeated tokens can be omitted altogether.

Technique A is to omit fully identical tokens.
For example,

```
/Object#1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
/Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.Key2 Value2
/Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.on+Xagn0071xQ
```

Can be compressed to:

```
/Object#1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
!1Cs7M+XaUth1_K.Key2 Value2
.on
```

Technique B is to omit repeating halves (values and origins) separately.
This necessitates an adjustment to the text-based serialization.
Transcendent values now must mention their 0 origin explicitly, e.g. `/Object+0`.

Technique A is the default behavior in case a deserializer encounters a token omission and there are no prior instructions.

In the case of binary coding, there is no way to omit tokens, but repeated tokens can be replaced with special values (e.g. 11111..11).
It is an open question whether it pays off, compared to straight gzip.

JSON based encoding benefits from shorter specs exactly the same way as the text-based encoding.
Compression does not span JSON frames.

