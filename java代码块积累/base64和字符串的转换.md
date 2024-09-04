
### base64转换为字符串

```
    /**
     * base64转字符串
     * @param base64
     * @return
     */
    public static String base64ToStr(String base64) {
        if (StringUtils.isBlank(base64)) {
            return StringUtils.EMPTY;
        }
        byte[] bytes = Base64.getUrlDecoder().decode(base64.getBytes(StandardCharsets.UTF_8));
        return new String(bytes, StandardCharsets.UTF_8);
    }
```



### 字符串转换为base64

```
    /**
     * 字符串转base64
     * @param str
     * @return
     */
    public static String strToBase64(String str) {
        if (StringUtils.isBlank(str)) {
            return StringUtils.EMPTY;
        }
        byte[] bytes = Base64.getUrlEncoder().encode(str.getBytes(StandardCharsets.UTF_8));
        return new String(bytes, StandardCharsets.UTF_8);
    }

```
