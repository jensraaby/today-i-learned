# Creating a Finagle HTTP client with a P12 client certificate for TLS based authentication

At the BBC we use our public key infrastructure (PKI) to control HTTP access to our backend systems over TLS (SSL). 

This means when you use an HTTP client, you need to provide client SSL configuration so that the server will be able to authenticate your request.

This is all fine and dandy with many libraries (e.g. Python's requests), but with some JVM implementations it's a bit more work.


At the moment I'm working with Finagle for simple HTTP integration, but it doesn't provide a simple way of setting up a TLS client certificate.

There are several pieces of the puzzle. Ultimately Finagle lets you supply a `Netty3TransporterTLSConfig` to your client. Why does it have a name like that?
Well, under the hood Finagle uses Netty for IO, and I suspect it is easier to delegate a lot of the HTTPS/TLS protocol work to that library rather than wrap it (as with the majority of the Finagle HTTP library).

A `Netty3TransporterTLSConfig` takes 2 arguments in its constructor - a function of type `SocketAddress => Engine` and an `Option[String]` with the hostname to verify against the server certificate. See the source code here:
https://github.com/twitter/finagle/blob/develop/finagle-core/src/main/scala/com/twitter/finagle/netty3/Netty3Transporter.scala

The function should essentially return a new SSL Engine for the request. I could not find any way to create a socket-specific Engine, but I am sure there is a reason for this as it is mentioned in the Finagle changelog.


If you use PEM files to store certificates, there is a provided method to convert them to Java Key Store files, see: https://github.com/twitter/finagle/blob/develop/finagle-core/src/main/scala/com/twitter/finagle/ssl/PEMEncodedKeyManager.scala


So, if you just want to use your P12 file (with a password) directly, here are the magic incantations (obviously you'll need to customise bits of this):
```
import java.io.InputStream
import java.security.KeyStore
import javax.net.ssl.{KeyManagerFactory, SSLContext}

val pathToP12 = "/path/to/a/p12"
val keyPassword = "my$3cr3t"

val pathToTrustStore = "/path/to/certs.jks"
val trustStorePassword = "some-password-probably-changeit"

def sslContext = {
    // load the P12 from disk and decrypt it into a KeyStore
    val keyStoreInputStream = new FileInputStream(new File(pathToP12))
    val ks = KeyStore.getInstance("PKCS12")
    ks.load(keyStoreInputStream, keyPassword.toCharArray)
   
    // a KeyManagerFactory takes the KeyStore and can provide an array of KeyManagers
    val kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm)
    kmf.init(ks, keyPassword.toCharArray)
    
    // similarly to the key, we need to load a TrustStore to verify the server certificate
    val trustStoreInputStream = new FileInputStream(new File(pathToTrustStore))
    val trustStore = KeyStore.getInstance("JKS")
    trustStore.load(trustStoreInputStream, trustStorePassword.toCharArray)
    
    val tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm)
    tmf.init(trustStore)
    
    // the sslContext needs the array of KeyManagers and TrustManagers
    val tlsSslContext = SSLContext.getInstance("TLS")
    tlsSslContext.init(kmf.getKeyManagers, tmf.getTrustManagers, new SecureRandom())
    tlsSslContext
}

val client = Http.client
		.withTransport.tls(sslContext)
		.newService("some-host-name:8080")

```

This is not that pretty - it is the sort of thing I wish all HTTP libraries would abstract away (Python requests does, and the Play Framework does a pretty good job too).
