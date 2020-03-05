---
num: Lecture 34
lecture_date: 2020-03-05
desc: "GET vs POST vs. all the others"
ready: false
pdfurl:
---

# The HTTP protocol

HTTP`, and it's secure cousin `HTTPS`, are *protocols* for communication between
* a web client (e.g. a browser, but also programs that are fetching data from APIs)
* a web server (anything that listening on an address that starts with `http://` or `https://`

HTTP is a "request/response" protocol.

* Requests have *method* types
   * GET
   * POST
   * many others
* Responses have response codes
   * 200 (success)
   * 404 (not found)
   * many others...
   
One key to understanding web interactions is understanding a bit about web requests, responses and 
status codes.


# HTTP methods GET and POST

*Not to be confused with methods of a class in Java, or any other object-oriented language.)*

The main two HTTP methods are GET and POST.
* These are the only two that you'll commonly see coming from a `<form>` element in a browser.


GET:
* is typically used for just retrieving information, when nothing changes on the server
* puts all the parameters in the URL where they can be seen
   * pro: requests can be bookmarked along with their paramters
   * con: not so good if the params are long, or contain sensitive data

POST:
* is typically used when something is going to CHANGE on the server
   * e.g. you are going to update a database, or some other change to server state in some way
   * also used for login/logout because that changes the state of the users's session, which is stored on the server
* often has CSRF protection enabled; see Scott Chow's writeup here: <https://ucsb-cs56.github.io/topics/spring_boot_post_and_csrf/>

# Example of logout form using POST from lab07

* This [excerpt from lab07](https://github.com/ucsb-cs56-w20/STARTER_lab07/blob/f437a29da7534058c9edfc39ab00f07a1d479d25/src/main/resources/templates/bootstrap/bootstrap_nav_header.html#L36-L39)
  shows the Spring Boot CSRF protection in a logout form.
  ```html
  <form method="POST" action="/logout" class="form-inline" th:if="${isLoggedIn}">
    <button class="navbar-btn" type="submit">Logout</button>
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
  </form>
  ```

# For more detail

* <https://www.freecodecamp.org/news/http-and-everything-you-need-to-know-about-it/>

