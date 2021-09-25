




```JAVA
package com.example.demo;


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.List;

@SpringBootTest
public class JsonBean {


    /**
     * JSON字符串
     */
    private static final String jsonStr = 
                        "{\"retCode\": 200, " +
                        "\"retMsg\": \"success\", " +
                         "\"retData\": " +
                                "{\"result\": " +
                                 "[" +
                                        "{\"tag\": \"demo\", " +
                                        "\"xmin\": 80, " +
                                        "\"ymin\": 1600, " +
                                        "\"xmax\": 1650," +
                                        " \"ymax\": 1754}], " +
                                  "\"my_url\": \"http://www.baidu.com\"}}";


    /**
     * JSON字符串对应的java实体类
     */
    @Data
    public static class ResultResp{
        private Integer retCode;
        private String retMsg;
        private RetData retData;

        @Data
        public static class RetData{
            private List<Result> result;
            @JsonProperty("my_url")
            private String myUrl;
        }

        @Data
        public static class Result{
            private Integer xmin;
            private Integer xmax;
            private Integer ymin;
            private Integer ymax;
            private String tag;
        }
    }


    /**
     * 1.jackson：json字符串 与 java对象 互相转化
     * @throws IOException
     */
    @Test
    void test01() throws IOException {
        // 1.将json字符串转成java对象
        ObjectMapper objectMapper = new ObjectMapper();
        ResultResp resultResp = objectMapper.readValue(jsonStr, ResultResp.class);

        // 2.将java对象转成json字符串
        List<ResultResp.Result> result = resultResp.getRetData().getResult();
        String myUrl = resultResp.getRetData().getMyUrl();
        System.out.println(myUrl);
        
        String stringJsonresult = objectMapper.writeValueAsString(result);
        System.out.println(stringJsonresult);
    }


    /**
     * 2.fastjson：
     *      2.1 json字符串 与 java对象互相转化 ； 
     *      2.2 json字符串 与 json对象互相转化
     * 
     * @throws IOException
     */
    @Test
    void test02() throws IOException {
        // 1.将json字符串转成json对象 然后从json对象中获取数据
        JSONObject jsonResultResp = JSON.parseObject(jsonStr);
        JSONObject retData = jsonResultResp.getJSONObject("retData");
        JSONArray result = retData.getJSONArray("result");
        System.out.println(result.toJSONString());

        System.out.println("=======================================");

        // 2.将json字符串转成java对象 然后将java对象转成json字符串传给前端
        ResultResp resultResp = JSON.parseObject(jsonStr, ResultResp.class);
        List<ResultResp.Result> result1 = resultResp.getRetData().getResult();
        String stringJsonresult = JSON.toJSONString(result);
        System.out.println(stringJsonresult);
    }
}



```


