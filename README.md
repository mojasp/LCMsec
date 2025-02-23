LCMsec - Security for the Lightweight Communications and Marshalling Protocol (LCM)

LCM is a set of libraries and tools for message passing and data marshalling,
targeted at real-time systems where high-bandwidth and low latency are
critical. It provides a publish/subscribe message passing model and automatic
marshalling/unmarshalling code generation with bindings for applications in a
variety of programming languages.

This fork, LCMsec, is a security extensions for LCM. It:
* provides confidentiality, integrity and authenticity to LCM messages
* provides an attribute-based acces control mechanism with multicastgroup, port and channel granularity
* aims to minimize both overhead and computational complexity - as such, LCMsec is designed to be usable in most environments in which LCM is used.
* maintains the decentralized, peer-to-peer nature of LCM: no broker, no central instance to facilitate keyexchange
* is scalable to many peers and capable of dealing with dynamic communication topologies (hosts leaving and joining a channel) efficiently

This is achieved with the use of:
* a hybrid cryptosystem: a symmetric key is shared between all publishers and subscribers to a channel on a specific multicastgroup
* No distinction between subscribers and publishers, each subscriber is also allowed to publish messages. This makes it possible to use a single shared symmetric key, and avoids issues typically associated with multicast authentication
* A built-in, decentralized discovery protocol
* AES/GCM or ChaChaPoly1305 as the underlying authenticated encryption algorithm.
* The Dutta-Barua group key agreement (modified to use EC-Cryptography) to agree on the symmetric key while minimizing the number of network interactions when a publisher or subscriber joins a topic (i.e., is turned on)

For more informations please take a look at the [paper on LCMsec](https://ieeexplore.ieee.org/abstract/document/10333516).

# Limitations of LCMsec

**The Implementation of LCMsec has not been vetted independently. Use at your own risk.** 

* Additional dependencies are Botan and libstdc++.
* Not yet adressed are the issues of rekeying and certificate revocation, which will be a next step. 
* Only C, C++ and Python bindings are available, Python bindings have not been tested recently
* lcm-logger and lcm-spy are not availe
* Only the udpm provider is available

# LCMsec Quickstart guide

* To use security features, you will need generate a root certificate and x509v3 certificate/key-pairs for each node that should communicate. Permissions are written into SAN of those certificates. For a way to generate certificates automatically, refer to `examples/cpp_security/gen_certificates.py`
* The format of the SAN is a URI of the form: `urn:lcmsec:gkexchg:<mcastgroup>:<port>:<channelname>:<user_id>` 
    * example: ` urn:lcmsec:gkexchg:239.255.76.67:7667:channel1:4` (for a channel)
    * for the permission to be part of a multicast group, use the special string "gkexchg_g" and omit the channelname, for instance: ` urn:lcmsec:gkexchg_g:239.255.76.67:7667:4`
* use ```lcm_create_with_security(...) ``` (or use the c++ API in lcm-sec.hpp) to create an LCM instance that has security enabled. This call will block until the group key agreement has finished on all channels for which there are permissions in the certificate
* At this point, just use the created lcm instance as you normally would.

An Example for an application using LCMsec can be found in `examples/cpp_security` folder.
