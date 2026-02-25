# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```python

# Thrift schema (student.thrift)
%%writefile ../schema/person.thrift
struct Student {
1:required string name,
2: required i32 age,
3: required list<string> courses
}
Service school {
Student enrollCourse(1: required Student student, 2: required string course)
}


# Thrift server (student_server.py)
%%writefile ../student_thrift_server.py
import thriftpy2
from thriftpy2.rpc import make_server

# Load the Thrift schema
student_thrift = thriftpy2.load("./schema/student.thrift", module_name="student_thrift")

class SchoolHandler:
    def enrollCourse(self, student, course):
        # Add the course into the list, then return the updated record
        student.courses.append(course)
        return student

server = make_server(student_thrift.School, SchoolHandler(), client_timeout=None)

print("Starting and running the server...")
print("Press Ctrl+C to stop the server.")
server.serve()


# Thrift client (student_client.py)

```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```python
# Protobuf schema (book.proto)
%%writefile ../schema/book.proto
syntax = "proto3";

message Book {
  string title = 1;
  string author = 2;
  int32 page_count = 3;
}

service Library {
  rpc checkoutBook(Book) returns (Book) {}
}



# Protobuf server (book_server.py)

%%writefile ../book_server.py
from concurrent import futures
import grpc

#Generated gPRC code from the book.proto file
import book_pb2
import book_pb2_grpc

class Library(book_pb2_grpc.LibraryServicer):

    def checkoutBook(self, request, context):
        # request is a Book message sent from the client
        # The question says: return the same Book message
        return request

#create the gRPC server and add the Library service to it
server = grpc.server(futures.ThreadPoolExecutor(max_workers=2))
#Register the Library service with the server
book_pb2_grpc.add_LibraryServicer_to_server(Library(), server)
server.add_insecure_port('[::]:50051')
print("Starting and running the server...")
print("Press Ctrl+C to stop the server.")
#start the server and keep it running until interrupted
server.start()
server.wait_for_termination()

# Protobuf client (book_client.py)
import sys
sys.path.append('..') #allow importing generated files from project root

import grpc
import book_pb2
import book_pb2_grpc

#Helper function to call the checkoutBook method on the server
def checkout_book(stub, book):

    #call the RPC method and returns a book
    book = stub.checkoutBook(book)
    return book

with grpc.insecure_channel('localhost:50051') as channel:
    # Create a Book message to send to the server
    my_book = book_pb2.Book(title="DSAI", author="ElaineT", page_count=96)

    #Create a stub (client) for the Library service
    stub = book_pb2_grpc.LibraryStub(channel)

    #Call the checkoutBook method and print the returned Book message
    returned = checkout_book(stub, my_book)

    # print result to show it is returned correctly
    print(returned)
```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.
