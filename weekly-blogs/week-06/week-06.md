# week-06

## What I met

After my repeated attempts, I found that the members of rustls were not very willing or energetic to communicate about the replacement of the ring backend. Because I did not get a reply to my email, comment (see <https://github.com/rustls/rustls/issues/521>). But once I filed a small PR(see <https://github.com/rustls/rustls/pull/1085>), I soon received feedback. I think maybe I should just open up a new issue to draw their attention.

So after a discussion with Vivian, I think I would begin my work breaking changes fork without their blessing.

Here is how I think the document is structured.

```shell

project name: wasmedge-rustls (a fork of rustls)
.
├── ...                 # The project structure remains the same, but all ring and webpki calls will be replaced
├── src
│ └── ...
│─ tests
│ └── ...
│── crypto-traits/*     # Contains traits that replace the ring encryption algorithm
│── crypto-impls/*      # The implementation of the corresponding traits, in this section I will use wasi-crypto as a dependency
└── ...

```

## What problems I had

My problems at this stage are two main points.

- I'm quite unsure because I can't get advice from the official members of rustls.
- I'd like to know how to test HTTPS support, i.e. what should my Use case look like?

Specifically, what are some examples of wasmedge calls to TCP and HTTP at this stage? I've searched the rust documentation and there is no rust networking material. How should I test HTTPS support maybe WasmEdge's example of using TCP and HTTP support can help me.

## What's next

- I will just open up a new issue to draw rustls folk's attention.
- Set up my fork branch with a trying of simple trait extraction.
- Figure out how to test HTTPS in WasmEdge or how we wish it to be like.
