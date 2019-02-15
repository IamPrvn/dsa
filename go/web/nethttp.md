[TOC]





# net/http

- The net/http library is divided into two main parts :-

  - client
  - server

  various structs supporting each other are as following :

![](https://www.safaribooksonline.com/library/view/go-web-programming/9781617292569/03fig01_alt.jpg)

- handling request with go server 

  ![img](https://www.safaribooksonline.com/library/view/go-web-programming/9781617292569/03fig02_alt.jpg)



- A simple go server :-

  ```go
  package main

  import "net/http"

  func main() {
    // 1st arg is address, second argument is handler. If the network address is an 	// empty string, the default is all network interfaces at port 80. If the handler   // parameter is nil, the default multiplexer, DefaultServeMux, is used.      
    http.ListenAndServer("", nil)  
  }
  ```

- A simple server does not take much configuration, so use `server` struct :

  ```go
  type Server struct {
      Addr           string
      Handler        Handler
      ReadTimeout    time.Duration
      WriteTimeout   time.Duration
      MaxHeaderBytes int
      TLSConfig      *tls.Config
      TLSNextProto   map[string]func(*Server, *tls.Conn, Handler)
      ConnState      func(net.Conn, ConnState)
      ErrorLog       *log.Logger
  }
  ```



## tls go server

- First create a x509 certificate template.

  ```go
  import (
    "crypto/rand"
    "crypto/x509"
    "crypto/x509/pkix"
    "math/big"
    "time"
  )

  max :=  new(big.Int).Lsh(big.NewInt(1), 128) //Left shift the big no by 128
  serialNumber, _ := rand.Int(rand.Reader, max) //rand.Reader is pseudorandom random 												// no generator ..uses /dev/urandom

  subject := pkix.Name {
    Organization:
    OrginaztionalUnit:
    CommonName:
  }

  template := x509.Certificate{
    SerialNumber: serialNumber,
    Subject: subject,
    NotBefore: time.Now(),
    NotAfter: time.Now().Add(365*24*time.Hour),
    Keyusage: x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
    ExtKeyUsage: []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
    IPAddresses: []net.IP{net.ParseIP("127.0.0.1")},
  }
  ```

- Next, we generate the private key

  ```go
  import "crypto/rsa"

  pk , _ := rsa.GenerateKey(rand.Reader, 2048)
  ```

  ```go
  /*
  type PrivateKey struct {
  	PublicKey            // public part.
  	D         *big.Int   // private exponent
  	Primes    []*big.Int // prime factors of N, has >= 2 elements.

  	// Precomputed contains precomputed values that speed up private
  	// operations, if available.
  	Precomputed PrecomputedValues
  }
      A PrivateKey represents an RSA key
  */
  ```

- The RSA private key struct that’s created has a public key that we can access, useful when we use the `x509.CreateCertificate` function to create our SSL certificate :

  ```go
  //func CreateCertificate(rand io.Reader, template, parent *Certificate, pub, priv interface{}) (cert []byte, err error)
  ```


  derBytes, _ := x509.CreateCertificate(rand.Reader, &template, &template, &pk.PublicKey, &pk)
  ```

- Now encode the der bytes in cert.pem, similarly encode the privatekey to in PEM format

  ```go
  certOut, _ := os.Create("cert.pem")
  pem.Encode(certOut, &pem.Block{Type: Certificate, Bytes: derbytes})
  certOut.Close()

  keyOut, _ := os.Create("key.pem")
  pem.Encode(keyOut, &pem.Block{Type: "RSA Private Key", Bytes: x509.MarshalPkcs1PrivateKey(pk)})
  keyout.Close()
  ```

- Now use this certOut, and keyOut as :

  ```go
  http.ListenAndServeTLS("cert.pem", "key.pem")
  ```



## handler and handler functions

### handlers

- ```go
  type Handler interface {
    ServeHTTP(w http.ResponseWriter,r *http.Request)
  }

  //DefaultServeMux is an instance of ServeMux and ServeMux implements the ServeHttp method. So DefaultServeMux is also an instance of Handler.
  ```

- Let's create our own handler

  ```go
  package main

  import (
    "net/http"
    "io"
  )

  type MyHandler struct {}

  func (h * Myhandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "Hello World")
  }
  ```


  func main() {
    myhandler := MyHandler{}
    server := http.Server{
      Addr: "127.0.0.1:8080",
      Handler: &myhandler,
    }
    server.ListenAndServe()
  }
  ```

  Here’s the tricky bit: if you go to http://localhost:8080/anything/at/all you’ll still get the same response! Why this is so should be quite obvious. We just created a handler and attached it to our server, so we’re no longer using any multiplexers. This means there’s no longer any URL matching to route the request to a particular handler, so all requests going into the  server will go to this handler.**** 

  **Thats why multiplexer are used**.  Most of the time we want the server to respond to more than one request, depending on the request URL.

- Most of the time, we don’t want to have a single handler to handle all the requests instead we want to use different handlers instead for different URLs. **To do this, we don’t specify the Handler field in the Server struct (which means it will use the DefaultServeMux as the handler); we use the http.Handle function to attach a handler to DefaultServeMux. Notice that some of the functions like Handle are functions for the http package and also methods for ServeMux.** These functions are actually convenience functions; calling them simply calls DefaultServeMux’s corresponding functions. If you call http.Handle you’re actually calling DefaultServeMux’s Handle method.      

  ```go
  package main

  import (
    "net/http"
    "io"
  )

  type HelloHandler struct {}

  func (h * HelloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "Hello")
  }

  type WorldHandler struct {}

  func (h * WorldHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "World")
  }


  func main() {
    hello := HelloHandler{}
    world := WorldHandler{}
    server := http.Server{
      Addr: "127.0.0.1:8080",
    }
    
    http.Handle("/hello", &hello) // it simply calls SimplServeMux.Handle()
    http.Handle("/world", &world)
    
    server.ListenAndServe()
  }
  ```

  > Notice that some of the functions like `Handle` are functions for the http package and also methods for ServeMux. These functions are actually convenience functions; calling them simply calls DefaultServeMux’s corresponding functions. If you call http.Handle you’re actually calling DefaultServeMux’s Handle method. 

### handler functions

- You do not need to create handler evrytime,  creating handler functions will also work. Handler functions are functions that behave like handlers. Handler functions have the same signature as the ServeHTTP method; that is, they accept a ResponseWriter and a pointer to a Request.

- Let's convert the above program to use handle functions instead of handlers :-

  ```go
  package main

  import (
    "net/http"
    "io"
  )

  func hello(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "Hello")
  }

  func world(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "World")
  }
  ```


  func main() {
    server := http.Server{
      Addr: "127.0.0.1:8080",
    }
    
    http.HandleFunc("/hello", hello) // it simply calls SimplServeMux.HandleFunc()
    http.HandleFunc("/world", world)
    
    server.ListenAndServe()
  }

  **How does it work ?**

  Let us see the source code of `HandleFunc` 

  ```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {     		   		DefaultServeMux.HandleFunc(pattern, handler)
  }

func (mux *ServeMux) HandleFunc(pattern string, 
                                handler func(ResponseWriter,*Request)) {
     mux.Handle(pattern, HandlerFunc(handler))
  }
  ```



### Why use handlers if handler functions fo the job?

1. It all boils down to design. If you have an existing interface or if you want a type that can also be used as a handler, simply add a ServeHTTP method to that interface and you’ll get a handler that you can assign to a URL. 
2. It can also allow you to build web applications  that are more modular.



## Chaining

### handler funtions

```go
func hello (w http.ResponseWriter, r *http.Response) {
  fmt.Fprintf(w, "hello World")
}

func log (h http.HandlerFunc) http.HandlerFunc {
  return func(w http.ResponseWriter, r *http.Response) {
    name := runtime.FuncForPC(reflect.ValueOf(h).Pointer()).Name()
    fmt.Println("Handler function called" + name)
    h(w, r)
  }
}

func protect(h http.HandlerFunc) http.HandlerFunc {
  return func(w http.ResponseWriter, r *http.Response) {
    ---
    h(w,r)
  }
}

func main() {
  server := http.Server {
    Addr: "127.0.0.1:8080",
  }
  http.handleFunc(protect(log(hello)))
  
  server.ListenAndServe()
}
```

### handlers

```go
type HelloHandler struct{}

func (h *HelloHandler) ServerHTTP(w http.ResponseWriter, r *http.Response) {
  fmt.Fprintln(w, "Hello")
}

func log(h http.Handler) http.Handler {
  return http.HandlerFunc( func(w http.ResponseWriter, r *http.Response) {
    ....
    h.ServeHTTP(w, r)
  })
}

func protect(h http.Handler) http.Handler {
  return http.HandlerFunc( func(w http.ResponseWriter, r *http.Response) {
    ...
    h.ServeHTTP(w, r)
  })
}
```



## ServeMux and DefaultSeveMux

- ServeMux is a struct with a map of entries that map a URL to a handler. It’s also a handler because it has a ServeHTTP method. ServeMux’s ServeHTTP method finds the URL most closely matching the requested one and calls the corresponding handler’s ServeHTTP
- DefaultServeMux is an instance of ServeMux that’s publicly available to the application that imports the net/http library. It is that same instance which is being used, when you pass a nil handler to `http.ServerAndListen`

> For any registered URLs that don’t end with a slash (/), ServeMux will try to match the exact URL pattern. If the URL ends with a slash (/), ServeMux will see if the requested URL starts with any registered URL.      