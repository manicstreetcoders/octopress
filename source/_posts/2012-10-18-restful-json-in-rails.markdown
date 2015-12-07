---
layout: post
title: "RESTful JSON in Rails"
date: 2012-10-18 10:10
comments: true
categories: 
---

* [Building a platform API on rails](http://blog.gomiso.com/2011/06/27/building-a-platform-api-on-rails)
* [If you're using to_json, you're doing it wrong way](http://engineering.gomiso.com/2011/05/16/if-youre-using-to_json-youre-doing-it-wrong)
* [RailsCasts #322 RABL](http://railscasts.com/episodes/322-rabl)
* [Rendering errors in json with rails](http://travisjeffery.com/b/2012/04/rendering-errors-in-json-with-rails)
* [RailsCasts #350 REST API Versioning](http://railscasts.com/episodes/350-rest-api/versioning)

을 번역 정리 해본다.

### Curl 로 JSON 서버를 Test 할 경우

`curl -v -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '[{"exam_id":1,"user":"GILDONG", "answer":1,"used_time":5.2 }]' http://localhost:3000/results`

* `-v` 로 verbose 세팅
* `-X POST` 로 HTTP METHOD 를 *POST* 로 세팅
* 그리고 *Accept:* 와 *Content-type:* 헤더를 *application/json* 으로 세팅

### Handling Errors

RESTful JSON 을 Rails 로 구현할때 HTTP Status Code 를 깜빡하기 쉽다.

    def create
      @model = Model.create(params[:model])
      if @model.save
        render :json => @model
      else
        render :json => { :errors => @model.errors.full_messages }, :status => 422
      end
    end

Error 시에 적절하게 HTTP Status Code 를 세팅하는 것. 예를 들어 *422 unprocessable entity* 로 세팅한다거나.

full_messages 대신에 errors 오브젝트를 사용할수도 있어.

    render :json => { :errors => @model.errors }

### Versioning REST API

### Securing API & OAuth

#### id:password 방식
[Railscasts ep352: Securing an API](http://railscasts.com/episodes/352-securing-an-api?autoplay=true) 를 보면

기본적으로는 Rails controller 에

    http_basic_authentication_with name: "dhh", password: "secret"

으로 하면 `curl http://localhost:3000/api/products` 로 하면 `401 Unauthorized`
되고, 
`curl http://localhost:3000/api/products -u 'admin:secret'` 처럼 id:password 를 주면 된다.

* [Designing a Secure REST API without OAuth](http://www.thebuzzmedia.com/designing-a-secure-rest-api-without-oauth-authentication/)
* Web API Design: Crafting Interfaces that Developers Love
* [gae9 API](https://github.com/ltbl/api.gae9.com)
