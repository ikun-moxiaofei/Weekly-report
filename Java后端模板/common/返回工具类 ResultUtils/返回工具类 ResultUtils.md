ResultUtils
```java
public class ResultUtils {

    /**
     * 成功
     *
     * @param data
     * @param <T>
     * @return
     */
    public static <T> BaseResponse<T> success(T data) {
        return new BaseResponse<>(0, data, "ok");
    }

    /**
     * 失败
     *
     * @param errorCode
     * @return
     */
    public static BaseResponse error(ErrorCode errorCode) {
        return new BaseResponse<>(errorCode);
    }

    /**
     * 失败
     *
     * @param code
     * @param message
     * @return
     */
    public static BaseResponse error(int code, String message) {
        return new BaseResponse(code, null, message);
    }

    /**
     * 失败
     *
     * @param errorCode
     * @return
     */
    public static BaseResponse error(ErrorCode errorCode, String message) {
        return new BaseResponse(errorCode.getCode(), null, message);
    }
}
```
ResultUtils 类是一个工具类，它提供了静态方法来创建 BaseResponse 对象。这些方法使得创建响应更加方便。
这些方法可以用于快速生成表示成功或失败的响应对象，这些响应对象可以用于API的返回结果或者其他的业务逻辑中。其中，成功响应的状态码为0，失败响应的状态码通常为一个非零的错误码。
