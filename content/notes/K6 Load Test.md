---
title: "K6 부하 테스트"
tags:
- load test
---

공식 페이지 : https://k6.io/

Grafana k6 is an open-source load testing tool that makes performance testing easy and productive for engineering teams. k6 is free, developer-centric, and extensible.

Using k6, you can test the reliability and performance of your systems and catch performance regressions and problems earlier. k6 will help you to build resilient and performant applications that scale.

k6 is developed by Grafana Labs and the community.

## Key Feature
- k6 is packed with features, which you can learn all about in the documentation. Key features include:

- CLI tool with developer-friendly APIs.
- Scripting in JavaScript ES2015/ES6 - with support for local and remote modules
- Checks and Thresholds - for goal-oriented, automation-friendly load testing

## 사용처 

k6 users are typically Developers, QA Engineers, SDETs, and SREs. They use k6 for testing the performance and reliability of APIs, microservices, and websites. Common k6 use cases are:

### Load testing
k6 is optimized for minimal resource consumption and designed for running high load tests (spike, stress, soak tests) .

### Performance and synthetic monitoring
With k6, you could run tests with a small amount of load to continuously validate the performance and availability of your production environment.

### Chaos and reliability testing
k6 provides an extensible architecture. You can use k6 to simulate traffic as part of your chaos experiments or trigger them from your k6 tests.

## 사용할 수 없는 경우 

k6 is a high-performing load testing tool, scriptable in JavaScript. The architectural design to have these capabilities brings some trade-offs:

### Does not run natively in a browser
By default, k6 does not render web pages the same way a browser does. Browsers can consume significant system resources. Skipping the browser allows running more load within a single machine.

However, with xk6-browser, you can interact with real browsers and collect frontend metrics as part of your k6 tests.

### Does not run in NodeJS
JavaScript is not generally well suited for high performance. To achieve maximum performance, the tool itself is written in Go, embedding a JavaScript runtime allowing for easy test scripting.

If you want to import npm modules or libraries using NodeJS APIs, you can bundle npm modules with webpack and import them in your tests.

## 설치 
Mac 에서는 아래와 같이 `brew`를 이용하여 설치 할 수 있다. 
```
brew install k6
```
이외 다른 OS는 [installation 페이지](https://k6.io/docs/getting-started/installation/ )를 참고하면 된다.

## 테스트 스크립트 작성 방법
https://k6.io/docs/getting-started/running-k6/ 

## 결과값 해석 
https://k6.io/docs/getting-started/results-output/ 

## 200 OK 이외의 값 
https://k6.io/docs/using-k6/checks/ 
about this https://github.com/grafana/k6/issues/1828 

## 결과 비쥬얼라이제이션 Grafana랑도 연결 가능 
https://k6.io/docs/results-visualization/grafana-cloud/

## Loki extension 과도 연결 가능 
https://grafana.com/docs/loki/latest/clients/k6/ 

## Postman에서 K6로 넘어오기
https://github.com/grafana/postman-to-k6 
https://k6.io/blog/load-testing-your-api-with-swagger-openapi-and-k6/

## 예제들
- 환경(env) 에 따라 다른 domain 사용 방법 : https://github.com/quiiver/merino-explorations/blob/b32f61f11f2a31fa281111ddceecc76f13f89b60/load-tests/test.js
- k6 제공하는 샘플 : https://github.com/grafana/k6/tree/master/samples/REST_API_testsuite
- redirect 테스트 : https://github.com/grafana/k6/blob/master/samples/redirects.js 
 
## k6 learn 
https://github.com/grafana/k6-learn

## awesome K6
https://github.com/grafana/awesome-k6 	