# week-05

## Feedback 1: What language option to choose: rust

I made these comparisons below.

1 from the base dependency.

With c++ I think I need to bind the libssl part of OpenSSL to WebAssembly and then rust bind it with OpenSSL to call it. However, the algorithms required by tls1.3 have been tested by rust crypto, while tls1.2 may be weak. If only tls1.3 is supported, the development process will be smoother with rust.

2 from workload.

My approach is to go through rustls project for where to use the ring and WebPKI, in fact, not as much as I thought. rustls project itself has the amount of code in about 30,000 lines, the ring may be about 8,000 lines. If the developer wants to bind CPP's OpenSSL then the code size is estimated to be about the same, but this is completely written by itself (expected to be 10,000 lines). That is to say, the amount of CPP code I expect is beyond rusts.

3 from the debugging process.

Rust compiled into wasm directly on the webassembly test will be much easier, while the host function debug will be a lot of trouble. Developers need to dbg wasmedge to run webassemblt file, which is not easy.

Conclusion: Cpp will be more complicated, and with my last post opinion in this issue, I think rust will be a better choice.

## Feedback 2: Project Design

My initial idea is shown below, but this is not my final version yet, because I can feel that this design will cause more changes to the upstream rustls, which is difficult.

! [uml](uml-1.png)

I've sent an email to the maintainer of rustls to discuss this. I expect there will be considerable benefit to both communities.

I plan to complete a more detailed design next week and get feedback from the rustls community.
