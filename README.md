A mirror for tinyhttpd

### Prepare 
Compile for Linux
```
 To compile for Linux:
  1) Comment out the #include <pthread.h> line.
  2) Comment out the line that defines the variable newthread.
  3) Comment out the two lines that run pthread_create().
  4) Uncomment the line that runs accept_request().
  5) Remove -lsocket from the Makefile.
```
The role of each function:

     Accept_request: handle from the socket to listen to an HTTP request, where a large part of the server can reflect the request process.
     Bad_request: return to the client This is an error request, HTTP status? 400 BAD REQUEST.
     Cat: read a file on the server write socket socket.
     Cannot_execute: The main processing occurred when executing the cgi program.
     Error_die: Write the error message to perror and exit.
     Execute_cgi: run cgi program processing, is also a main function.
     Get_line: read the line of the socket, the carriage return line, etc. are unified for the end of the line breaks.
     Headers: write the head of the HTTP response to the socket.
     Not_found: the main processing can not find the requested file when the situation.
     Seversfile: calls cat to return the server file to the browser.
     Startup: initializes httpd services, including the establishment of socket, binding port, to monitor and so on.
     Unimplemented: Returns to the browser that the received HTTP request is not supported by the method.
     Recommended source code read order: main -> startup -> accept_request -> execute_cgi, familiar with the main workflow and then carefully look at the source of each function.

     work process

     (1) server startup, in the specified port or random selection port binding httpd service.
     (2) received an HTTP request (in fact, listen to the port when the accpet), derived from a thread running accept_request function.
     (3) remove the HTTP request in the method (GET or POST) and url ,. For the GET method, if there are parameters, the query_string pointer points to the url Behind the GET parameter.
     (4) format the url to the path array, said the browser request server file path, in the tinyhttpd server file is in the htdocs folder. When the url to / end, or url is a directory, the default path in the path.html, said the visit to the home page.
     (5) If the file path is valid, for non-parameter GET request, directly output the server file to the browser, that is written to the socket in HTTP format, jump to (10). Other cases (with parameters GET, POST mode, url for the executable file), then call excute_cgi function implementation cgi script.
    (6) read the entire HTTP request and discard, if it is POST to find Content-Length. HTTP 200 status code written to the socket.
    (7) to build two pipes, cgi_input and cgi_output, and fork a process.
    (8) in the child process, the STDOUT redirect to the write end of cgi_outputt, STDIN redirect to the cgi_input read, close the cgi_input write and cgi_output read the end, set the request_method environment variable, GET Set the query_string environment variables, POST, then set the content_length environment variables, these environment variables are to call the cgi script, then run the program with execl cgi.
