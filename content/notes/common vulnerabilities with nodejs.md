---
title: "서버 개발하며 쉽게 놓치는 취약점 5가지"
tags:
- nodejs
- security
---
## 1.  Injection Attacks
- [[SQL Injection]]
- [[NoSQL Injection]]
- [[Common Injection]]
```javascript
app.get("/user", (req, res) => {
  const id = req.query.id;
  const query = `SELECT * FROM users WHERE id = ${id}`;
```
- 문제점 : query parameter `id`로 `1 OR 1=1`와 같이 오게되면 `SELECT * FROM users WHERE id = 1 OR 1=1` 처럼 쿼리문이 되어 전체 users를 return 할 위험이있다. 
- 개선 : prepared statement로 쿼리하고 `id` 체크 후 사용하도록 
	- user input validation 중요!
```javascript
app.get("/user", (req, res) => {
  const id = req.query.id;
  const query = "SELECT * FROM users WHERE id = ?";
  connection.query(query, [id], (error, results) => {
    if (error) {
      throw error;
    }
    res.send(results);
  });
});
```

## 2.  Cross-Site Scripting (XSS)
- 웹사이트를 통해 악의적인 스크립트를 삽입하여 공격.
- URL param으로 스크립트를 삽입하여, 스크립트가 실행되도록 유도 
```javascript
app.get("/", (req, res) => {
	const name = req.query.name; 
	res.send(`<h1>Hello, ${name}</h1>`); 
});
```
- 문제점 : param `<script>alert('XSS')</script>` 와 같이 들어오는 경우

## 3.  Denial-of-Service (DoS)

## 4.  Improper Authentication and Authorization

## 5.  Insecure Direct Object References


## References
- [# 5 Common Server Vulnerabilities with Node.js](https://blog.javascripttoday.com/blog/node-js-server-vulnerabilities/)
