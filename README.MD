## The task

Build a prototype of passwordless user log in program using Zero Knowledge Proof - Chaum Pedersen Protocol.

## System design

1. Communication protocol - use HTTP (or websocket or other standard protocols)

2. Server - REST API framework using Actix Web since it is a small, fast and powerful asynchronous Rust web framework for building APIs and web app.

3. Client - Any web client will be good: browser, Postman, curl etc, we will use curl for quick prototyping and testing.

## System parameters

This program use the following toy parameters to showcase the ZKP flow, in production we will first need to determine the security assumption before determining parameters, e.g. 128 bit security which essentially requires the use of an Elliptic Curve library to generate the below parameters.

`// prime base of modulus 
const G: u64 = 4;
// another prie base of modulus 
const H: u64 = 9;
// prime number of finite field
const P: u64 = 23;
// prime order of G, H mod P
const Q: u64 = 11;
`

## Major functions

1. A user can create a user by posting a id, username, y1, y2 from the secret x

2. After creation, a user can commit r1, r2 from a random secret k, and receive a challenge c from server

3. A user can post an id and s value (s = (k -c * x) mod Q ), if s is correct, the user will be authenticated and logged in, otherwise a failure message will be returned

4. A user can update his y1, y2 after creating an entry

5. A user can get his user information by calling get_ticket function

6. A user can delete his record by providing the correct user information of his own

7. Helper functions to generate random x, k, c, and compute s, these functions are provided to make the testers's life easier, they are not really needed, in fact it is recommended that the client program is doing its own random number generation and s computation

## How to test 

1. Run the server

cd /path/to/project_folder

cargo run

You should see the server log:

`listening on 127.0.0.1:8000
`

2. Use curl for the client test

curl is available for Ubuntu and MacOS out of box, if you don't have it, refer to this resource to [install curl](https://idratherbewriting.com/learnapidoc/docapis_install_curl.html)

- Creat a ticket 

`curl -XPOST 127.0.0.1:8000/tickets -H "Content-Type: application/json" -d '{"id":1, "username":"alan", "y1": 2, "y2": 3}'
`

- Commit a ticket

`curl -XPOST 127.0.0.1:8000/tickets/commit/1 -H "Content-Type: application/json" -d '{"id":1, "username":"alan", "y1": 2, "y2": 3, "r1": 8, "r2": 4, "c": 0}'
`
You will receive a data struct with a new challenge "c"

Now you can compute your own s = (k - c * x) mod 11

Or you can let the server compute s by giving the c value returned from the above step
Note: replace the below c with your actual c value

- Compute s

`curl -XPOST 127.0.0.1:8000/tickets/compute -H "Content-Type: application/json" -d '{"x":6, "k": 7, "c": 30, "s": 0}'
`
- Log in a user 

Using the s from the above function
Note: replace the below s with your actual s value

`curl -XPOST 127.0.0.1:8000/tickets/login/1 -H "Content-Type: application/json" -d '{"id":1, "s": 3}'
`

- Fetch a user information

`curl -XGET 127.0.0.1:8000/tickets/1
`

- Update a user information

`curl -XPUT 127.0.0.1:8000/tickets/1 -i -H "Content-Type: application/json" -d '{"id":1, "username":"alan", "y1": 2, "y2":3, "new_y1": 7, "new_y2": 9}'
`
- Delete a user from the system

`curl -XDELETE -i 127.0.0.1:8000/tickets/1 -H "Content-Type: application/json" -d '{"id":1, "username":"alan", "y1": 7, "y2": 9}'
`
- Fetch system parameters

`curl -XGET -i 127.0.0.1:8000/tickets/params
`

- Get some random numbers

`curl -XGET -i 127.0.0.1:8000/tickets/random
`

## Test data

Here are the test data used to verify the correctness of the implementation

`
g = 4
h = 9
p = 23
q = 11
x = 6
y1 = 4^6 mod p = 2
y2 = 9^6 mod p = 3
k= 7
r1 = 4^7 mod p = 8
r2 = 9^7 mod p = 4
c = 4
s = (k - c*x) mod q = (7 - 4*6) mod 11 = 5

r1 = (g^s*y1^c) mod p = (4^5 * 2^4) mod 23 = 8
r2 = (h^s*y2^c) mod p = (9^5 * 3^4) mod 23 = 4
`







