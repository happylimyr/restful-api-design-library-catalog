# RESTful API Library Catalog Design

## 1\. Functional / non-functional requirements

Functional:

* Managing authors, books, and categories in a library.  
* Searching and filtering books by title, author, category, and publication year.  
* Tracking available copies of books.  
* Viewing detailed information about each author and their books.  
* Create, read, update, and delete (CRUD) operations for books, authors, categories.  
* Filtering and pagination for large collections.  
* Authentication for modifying data (admin can add, edit, remove books).


Non-functional:

* Performance \- API should respond within 200ms for most requests.  
* Scalability \- should support growth in the number of books/authors.  
* Security \- JWT-based authentication for all modifying endpoints.  
* Reliability \- all operations should use proper HTTP status codes.  
* Maintainability \- follow REST standards and Richardson Maturity Model Level 3\.  
* Caching \- GET requests should be cacheable using ETag and Cache-Control.

## 2\. Model description

Author

| author\_id | INT, PK |
| :---: | :---: |
| first\_name | VARCHAR (100) |
| last\_name | VARCHAR (100) |
| birth\_year | INT |
| nationality | VARCHAR (100) |

Category

| category\_id | INT, PK |
| :---: | :---: |
| category\_name | VARCHAR (100) |
| description | VARCHAR (200) |

Book

| book\_id | INT, PK |
| :---: | :---: |
| title | VARCHAR (255) |
| isbn | VARCHAR (15) |
| publication\_year | INT |
| author\_id | INT, FK |
| category\_id | INT, FK |
| pages | INT |
| avilable\_copies | INT (or BOOL ?) |

BookdAuthors(manty-to-many)

| book\_id | INT |
| :---- | :---- |
| author\_id | INT |
| author\_order | INT |

## 

## 3\. Operations description

**Books**

| Method | Endpoint | Description | Auth | Pagination | Cached |
| :---: | :---: | :---: | :---: | :---: | :---: |
| GET | `/api/books` | Get list of all books (filterable, paginated) | No | Yes | Yes |
| GET | `/api/books/{id}` | Get details of a book by ID | No | No | Yes |
| POST | `/api/books` | Add new book | Yes  | No | No |
| PUT | `/api/books/{id}` | Update book info | Yes  | No | No |
| DELETE | `/api/books/{id}` | Delete book | Yes  | No | No |

Filter example: GET /api/books?authorId=1

**Authors**

| Method | Endpoint | Description | Auth | Cached |
| :---: | :---: | :---: | :---: | :---: |
| GET | `/api/authors` | List authors | No | Yes |
| GET | `/api/authors/{id}` | Get author details | No | Yes |
| POST | `/api/authors` | Add new author | Yes | **No** |
| PUT | `/api/authors/{id}` | Update author | Yes | **No** |
| DELETE | `/api/authors/{id}` | Delete author | Yes | **No** |

**Categories**

| Method | Endpoint | Description | Auth | Cached |
| :---: | :---: | :---: | :---: | :---: |
| GET | `/api/categories` | List all categories | No | Yes |
| GET | `/api/categories/{id}` | Get category by ID | No | Yes |
| POST | `/api/categories` | Add new category | Yes | No |
| PUT | `/api/categories/{id}` | Update category | Yes | No |
| DELETE | `/api/categories/{id}` | Delete category | Yes | No |

## 4\. Meaningful status codes

| Status Code | Name | Meaning / Usage |
| :---: | :---: | :---: |
| 200 OK | Success | The request was successful; resource returned or updated correctly. |
| 201 Created | Created | A new resource was successfully created (e.g., new book, author, or category). |
| 204 No Content | No Content | The operation was successful but there is no response body (e.g., delete). |
| 400 Bad Request | Invalid Input | The client sent malformed or invalid data (e.g., wrong field type, missing required field). |
| 401 Unauthorized | Unauthorized | Authentication is required or the JWT token is missing/invalid. |
| 403 Forbidden | Forbidden | The user is authenticated but does not have permission to perform this action (e.g., not an admin). |
| 404 Not Found | Resource Not Found | The requested resource (book, author, category) does not exist. |
| 409 Conflict | Conflict | The request could not be completed due to a conflict (e.g., duplicate ISBN or existing dependency). |
| 500 Internal Server Error | Server Error | An unexpected error occurred on the server side. |

## 5\. Authentication

Method: JWT (JSON Web Token)  
Header: Authorization: Bearer \<token\>  
Token Payload Example:

{  
  "userId": 1,  
  "username": "admin",  
  "role": "ADMIN",  
  "exp": 1720012800  
}

## 6\. Pagination with HATEOAS

GET /api/books?page=2\&size=5  
Responce:   
{  
  "page": 2,  
  "size": 5,  
  "totalPages": 10,  
  "totalItems": 50,  
  "books": \[ ... \],  
  "links": {  
    "self": "/api/books?page=2\&size=5",  
    "next": "/api/books?page=3\&size=5",  
    "prev": "/api/books?page=1\&size=5"  
  }}

## 7\. Caching

GET /books, /authors, /categories :  
Cache-Control: max-age=600  
Data changes rarely

POST/PUT/DELETE :	  
Not cached	  
State-modifying operations  
