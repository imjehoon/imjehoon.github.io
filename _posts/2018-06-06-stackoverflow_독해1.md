---
layout: post
title: "StackOverFlow 독해1"
tags: [StackOverFlow, 영어, tech english]
category: [영어공부]
comments: true
---

* 굉장히 주관적인 독해입니다. 오역 및 의역이 포함되어 있습니다.
* (stackoverflow글을 퍼왔을때 문제가 있을경우 연락 주시면 삭제 하도록 하겠습니다.)

오늘은 [stackoverflow QnA 원문](https://stackoverflow.com/questions/50722473/how-to-return-proper-statuscode-in-restful-post-method-restassured-test-doesnt) 을 독해 하겠다. 

## How to return proper statuscode in RESTful post method? restassured test doesn't work because of it
## 어떻게 하면 restfult Post method 방식에서 적절한 상태 코드를 리턴 할수 있을까?  안심할수 있는 테스트가 작동하지 않는다 이유때문에


```java
@POST
@Path("addcus")
@Consumes(MediaType.APPLICATION_JSON)
public Response addCustomer(Customer cus) {

String query = " insert into customer (name, address, postal_code, contactperson, phonenumber, e_mail) values(?, ?, ?, ?, ?, ?)";
try {
    PreparedStatement pst = ds.getConnection().prepareStatement(query);


    pst.setString(1, cus.getName());
    pst.setString(2, cus.getAddress());
    pst.setString(3, cus.getPostal_code());
    pst.setString(4, cus.getContactperson());
    pst.setString(5, cus.getPhonenumber());
    pst.setString(6, cus.getE_mail());
    //pst.execute();
    return Response.status(Response.Status.CREATED).entity(cus).build();
} catch (SQLException ex) {
    return Response.status(Response.Status.NOT_FOUND).entity(cus).build();
}


}



@Test
public void addCustomerTest() throws IOException {
//String json = readWholeFile("newCustomer.json");
JSONObject test = new JSONObject();

test.put("name", "john");
test.put("address", "teststreet 4");
test.put("postal_code", "6236test");
test.put("contactperson", "Frenkie test");
test.put("phonenumber", "659486755");
test.put("e_mail", "administration@testcompany.com");

given()
        .header("Content-Type", "application/json")
        .body(test.toJSONString()).
        when().
        post(uri + "/customer/addcus")
        .then()
        .assertThat()
        .statusCode(404);         
}
```
질문자의 코드는 위와 같다. 대략 보면 api test case를 작성하는 것 같은데, 항상 201 코드가 리턴되서 어떻게 하면 404 코드가 작성되냐고 질문하는 것 같다.
그럼 아래 질문에 대해 독해를 해보자.

원문 
> Hello people. So i got this post method that posts a customer entity into my database using restful. Now i need to make a test for it to validate that it works. So i tried to break it but i cant seem to figure out how to make this test properly. Now i seem to get statuscode 201 created ALWAYS even if i dont actually insert stuff into my database. This is probably because i always return that response code. Now i tested it and yes the test was green and gave 201 when it didnt insert stuff in my db. Now my question is how do i make it so that if it doesnt actually insert my customer object into the db it gives a 404 or something else. I purposely commented the pst.execute(); to test if my test would fail it did not...
plz help ive been at this for hours but i just cant seem to figure it out

독해
> 안녕 사람들. 나는  post method를 가져와서 that이하를 한다, resltful를 사용하여 내 db에 고객 테이블에 게시합니다. 나는 그게 잘 작동하는지 테스트를 만들어야 합니다. 그래서 나는 break되기를 시도했다, 그러나 나는 테스트를 정확히 수행하는 방법을 알아내지 못했다.  지금 상태코드를 201(created) 항상 받는다, 내 db에 insert 조차 일어나지 않아도. 
그것은 아마도 항상 response code를 리턴하기 때문이다.  지금 테스트를 해봐도 항상 테스트는 성공하고 201을 리턴하고있다 내 db에 아무것도 insert되지 않아도. 나의 질문은 어떻게 하면 db에 나의 고객 객체가 insert되지 않았을때 404를 주거나 다른 코드를 주는지이다. 나는 의도적으로 pst.execute();를 주석 처리 하였고 나의 테스트가 실패할것이라 생각하고...
도와주길 바란다 여러 시간을 소비했지만 그러나 나는 그것을 알아낼 수 없었다.

첫 독해인데 너무 긴 글을 해버렸다..
다음부터는 좀 더 쉬운 글을 해야겠다.

일단 위 내용에 대해 답변도 달려있지만.. 답변까지 독해할 기운이 없다.

이제 자바 코드 해설로 들어오면, 질문자는 자기 코드가 실패하여 201 코드가 아닌 다른 코드가 오길 원했지만 올린 코드를 보면 단지 주석을 처리한다고 하여 익셉션이 발생하지 않기 때문에 catch문에 걸리지 않고 그냥 201 코드가 리턴되게 되어있다.

```java
// 위에 생략
//pst.execute();
return Response.status(Response.Status.CREATED).entity(cus).build();
} catch (SQLException ex) {
    return Response.status(Response.Status.NOT_FOUND).entity(cus).build();
}
```

만약 질문에 내용대로 익셉션이 발생하여 404가 리턴되길 원한다면 아래와 같이 코드를 바꿔주면 된다.

```java
// 위에 생략
//pst.execute();
//return Response.status(Response.Status.CREATED).entity(cus).build();
throw new SQLException();  // sqlexception을 발생시켜 404가 리턴된다.
} catch (SQLException ex) {
    return Response.status(Response.Status.NOT_FOUND).entity(cus).build();
}
```

질문자가 pst.execute() 만 주석 처리 한다고 익셉션이 발생할 것으로 예상하는 것 같다.
추가적으로 위에는 가장 간단하게 404를 얻는 방법을 얘기 한것이고 실제로 db에 입력이 안되었을 경우 404가 리턴되게 하려면 다른 방식으로 해야 한다.

먼저 한가지 방안을 생각해 본다면 pst.execute(); 보다는 pst.executeUpdate(); 를 사용하여 실제 입력이 일어나면 int 값이 반환되어 그 값을 가지고 응답 값을 분기 태우면 될 것 같은데... 실제 입력이 일어낫을때 executeUpdate(); 반환값을 확인해 봐야 할 것 같다.


