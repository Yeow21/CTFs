

At its core, `**``**curl`**``** is designed to transfer data using various protocols such as HTTP, HTTPS, FTP, SCP, SFTP, and more.

syntax:
`curl options URL`

#### man page
https://linux.die.net/man/1/curl

Common usage is to transfer data from a URL

fetch web page:
curl https://example.com

fetch webpage with path
curl https//example.com/README

fetch html content:
curl https://www.geeksforgeeks.org

### Make http request with custom header
`curl -H "Host: a7660a357d14f9d69b800fdf0ae0abfe" http://127.0.0.1:80`

### http get request with params
curl "http://127.0.0.1:80?a=f6f7ab90b0b37cfdb294f32a6f6c2e1b"


#### urls can be written in sets:
curl http://site.{one, two, three}.com

__URLs with numeric sequence series can be written as:__ 

curl ftp://ftp.example.com/file[1-20].jpeg


****Progress Meter:**** curl displays a progress meter during use to indicate the transfer rate, amount of data transferred, time left, etc. 

curl -# -O ftp://ftp.example.com/file.zip
curl --silent ftp://ftp.example.com/file.zip



## Handling HTTP Requests Using curl Command

`` The ` ``**``**curl`**``** allows you to send custom HTTP requests with various methods such as GET, POST, PUT, DELETE, etc. For instance, to send a GET request:

curl -X GET https://api.example.com/resource

Similarly, to send a POST request with data:

curl -X POST -d "key1=value1&key2=value2" https://api.example.com/resource

In this example, the `**``**-d`**``** flag is used to specify data to be sent with the request.

**Download file:**
`curl -o file_name URL...`
`-o` - save file to local machine
-O - save file with name of url

**upload file**
`curl -T uploadfile.txt ftp://example.com/upload/`

**Handling auth**
`curl -u username:password https://example.com/api


### Post form data
`curl -X POST -d "a=dc72b632beb311238074aa2a128497c7" http://127.0.0.1:80
`-x POST` - specifies post request
-`d` - Sets post data as parameter and value


