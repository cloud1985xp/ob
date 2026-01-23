---
tags:
  - ruby
  - rails
  - language
created: 2024-01-01
updated: 2025-01-23
status: active
---


1. LLM
2. Passkeys
3. Performance / Scaling

----
# I. LLM
- Building LLM applications in Ruby - Andrei Bondarev
- Yet Another Ruby DSL for LLM - Delton Ding

----

Why (not) Ruby?

|Andrei|Delton|
|---------------------|------------------------|
|Monoliths are back in fashion|We only rewrite thins in Rust|
|Good graps of sound|Good to develop DSL, metaprogramming|
|OOP / Good software dev fundamentals|
|Ruby is similar to Python||


What is Generative AI / AI Background

Andrei||
- LLM
- The Problems
	- Hallucinations,
	- Outdated data,
	- Relevant knowledge is not used
- RAG
	- Vector Embedding
- AI Agent

Delton
- Encoder / Decoder. Then transformer model
	- https://magazine.sebastianraschka.com/p/understanding-encoder-and-decoder
- Unstable
- How to justify propositions
    - Appeal to Authority
    - Inductive Reasoning
    - Deductive Reasoning
- Embedding
	- https://github.com/ankane/neighbor
	- Prolog
		- https://www.ruanyifeng.com/blog/2019/01/prolog.html
		- https://iter01.com/550980.html
		- https://programmermagazine.github.io/201308/htm/article3.html
		- https://github.com/yet-another-ai/prologmqi.rb

## Building LLM applications in Ruby

- 基本上跟 ihower 那場講的東西差不多了
- 前面在講 AI Train how important until 2030/2060
### RAG

Tech for enhancing the accuracy and reliability of generative AI models

- Vector embedding
- Retrieve relevant document related
- Send to LLM go get answer back

### Vector embedding

- Tech. to represent data in an N-dimensional space
- LLM encode meaning behind tests in the embedding space or ‘latent space’

~ “vector search” or “semantic search” search by “meaning” (not keyword search)
- Similarity search

RAG Prompt

Tip:

- instruction s to enforce a format or style of response

prompt file ⇒ template (.yml)

LLM synthesize the answer

Optimizing RAG

- Human eval (upvote or downvote +1 or -1)
- RAGAS metrics
    - Faithfulness: ensuring retrieved context can acts as a justification for the generated answer
    - Context Relevance: context is focused, with little to no irrelevant information
    - Answer Relevance: the answer addresses the actual question
    - https://docs.ragas.io/en/latest/concepts/metrics/index.html

### AI Agent

- Autonomous (semi-autonomous) general purpose LLM-powered programs
- Can use Tools(APIs, other systems)
- Work best with powerful LLMs
- Can be used to automate workflows/business processes and execute multi-step tasks

Ref: Microsoft: jarvis

# II. Let's Pop into Passkeys

[https://speakerdeck.com/heliocola/lets-pop-into-passkeys](https://speakerdeck.com/heliocola/lets-pop-into-passkeys)

gem “WebAuthn”

http://github.com/cedarcode/
https://github.com/cedarcode/webauthn-ruby

## What are passkeys

- Passkeys, replacement for passwords
- Web authentication (WebAuthn) standard, created W3C and FIDO alliance

Passkey is a public/private key pair protected by your device biometrics used for challenge based authentication.

More: 
A passkey is a FIDO-based, **phishing-resistant credential** to replace passwords.  
1. It can be hardware tokens or security keys, such as USB or Bluetooth devices.      
2. It utilizes asymmetric public-key cryptography for enhanced security.      
3. It’s flexibility to link with platforms, security keys, or sync across devices.

- FIDO alliance
	- **an organization** for providing open-source and secure passwordless authentication standards, including UAF, U2F, and FIDO2.
- FIDO2
	- **a set of standards** for secure online authentication developed by FIDO Alliance. FIDO2 comprises two main components: WebAuthn for passwordless logins and CTAP for secure device communication.
- [WebAuthn](https://medium.com/webauthnworks/introduction-to-webauthn-api-5fd1fb46c285)
	- **a JavaScript API** developed by the W3C and FIDO Alliance, empowers **web applications** authentication with FIDO2 standards. Passkey is one of the authentication methods WebAuthn supports.
- [Challenge-based Authentication (Response)](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)
	- https://www.geeksforgeeks.org/challenge-response-authentication-mechanism-cram/
	- https://www.arkoselabs.com/explained/challenge-response-authentication/
	- 
	- Domain ownership verification(DNS challenge)
	- Captcha


## WebAuthn

https://webauthn.guide/
https://medium.com/webauthnworks/introduction-to-webauthn-api-5fd1fb46c285

### Main Entities of WebAuthn
- User: you
- User Agent: browser
- Relying Party: The web application, (or mobile app .. etc)
- Authenticator: Your iPhone or other hardware component that processed verifying the user's identity, usually device-based biometrics or PIN, etc.
	- Ex: iPhone will use iCloud to store the passkeys you generated.
		- Can I use passkey to protect the iCloud account(Apple ID)?

public key cryptograpthy

A secret stored on one’s devices, unlocked with biometrics


Public and private key pair

To use it, your device will first execute a biometrics verification

User is asked to sign with private key

Web app/site checks with users’ public key

How it works

### Enroll Workflow(sign up)

Relying Party(your web application), when user ask to signup, Your web application:
- Generate WebAuthn User ID
- Load your app WebAuthn settings
- Create a Challenge
- Return a json back to user/browser (specific json for challenge)
    - base on your ruby app settings, like timeout, keyCredParams(ex: algorithm number), extensions, authenticatorSelection
- Created for this user’s session, the challenge is used,
- In order to ask user side to send public key to the app(server)

(store challenge in session?)

User side:
- Received the challenge the server responded, want to use the challenge to create passkeys.
- User create a passkeys (through agent and authenticator, with face id then create passkeys) (and sync to cloud account provider, the depends on the authenticator)
- Got passkeys and send data(with public key and raw data about challenge) back to app(server)

Server

- Verify the data with the challenge from the first step
    - valid type? valid id? , verify data
- Creating user record
    - Store
        - external id (webauthn credential raw_id)
        - Public key
        - sign_count (provide by webauth credential)
- create its passkeys
- Return a success response

### Authenticate Workflow(sign in)

- User ask for sign in, the Server generate challenge respond to user(agent)
- Use agent process challenge with Authenticator (with biometrics and get passkeys)
- The Authenticator create signature with private key return to user agent
- User agent(browser) Send signature to server
- Server compare digital signature (with public key), is valid. you are authenticated

Github Repo:
- webauthn-with-devise
- ruby-passkeys

### References:
- https://blog.logto.io/web-authn-and-passkey-101/
- Challenge Response Authentication Mechanism (CRAM)
	- https://www.geeksforgeeks.org/challenge-response-authentication-mechanism-cram/
	- https://www.arkoselabs.com/explained/challenge-response-authentication/

# III. Performance / Scaling

- [A Rails performance guidebook: from 0 to 1B requests/day](https://www.youtube.com/watch?v=mJw3al4Ms2o) 
- Handling 225k requests per second to RubyGems.org 

## A Rails performance guidebook: from 0 to 1B requests/day 
Cristian Planas
### Is Rails Scalable?

- Yes
	- ActiveScale?
	- But It's not all about Rails(Application)

Good Mindset face to server load, prerequisite
- Error budget / Embarace failure
- Monitoring



Performance? not all about Rails
Almost talks are about DB
- DB Index
- Sharding
- Greedy Select
- Optimize the Complex DB query
	- Pareto principle, the "vital few"
	- https://en.wikipedia.org/wiki/Pareto_principle
- Rapid dataset size. Example: masterdata, translations
- Denormalize Data Writing Cache against to DB
- Cold Storage

What is a good sever team
- Ownership
- Accountable
- High trust
- Embracing uncertainty

Ref: https://www.infoq.com/news/2023/11/high-performing-software-team/
- Purpose creates autonomy 
- Decentralized decision-making fuels empowerment
- High trust with psychological safety accelerates cohesion
- Embracing uncertainty sustains growth

Conclusion
- Mindset
- - tradeoff, no system perfect
- An exclusive pursuit of perfection can be counterproductive

Q&A
- When to start considering scaling/performance, like sharding? coldstorage? query performance?

## Handling 225k requests per second to RubyGems.org 
Samuel Giddins

```
Good artist copy
Great artist steal,
And smart engineer done no work!

-- The fastest work you can do is no work at all
```

### Profiling about rubygems.org

### Accident
RPS: 20k -> 225k

### Lessons:

- Don’t serve traffic :joy
- If you have to serve the traffic, let someone else do it
- Please don’t DDoS yourself (don’t help user DDoS your self)
- Find appropriate place to do diff kinds of work, ex: CDN, S3
- Do all the “normal” optimization work
- Use rails caching
- Add indices to your DB tables
- Set cache-control headers
- Scale up your DB/web servers
- Add rate limiting
- Be very grateful for sponsor who provide those services for free
- Use true but shocking & misleading talk titles :joy
- Ask very politely for people to not install gems when I have brunch plans :joy




## Building LLM applications in Ruby

Andrei Bondarev, vedio talk

github: langchainrb

前面在講 AI Train how important until 2030/2060


# Yet Another Ruby DSL for LLM

Delton Ding

### LLM

Attention Is All you Need, paper for AI background

Encoder / Decoder architecture . The transformer model .. etc

In the transformer model

it provide layer of “Multi-head Attention”

Attention(Q, K, V) = sofrtmax(QK/dk)V …

Many LLM model based on it, but when you read their paper you found they are very unstable

### LLM Application

ex: Langchain , framework for python

### Racing the Tokens

- The facts should be inferred, not guessed
    - Give huge background (to AI) about the problem you are getting to solve
- How to justify propositions
    - Appeal to Authority
    - Inductive Reasoning
    - Deductive Reasoning
- The Rule Matters

### Why Ruby

we only rewrite things in rust :joy

DSL, metaprogramming

## The Epic Hacks

1. performance tricks

Ticketed Request (Asynchorouns process)

2. Embedding

text2vec-base-multilingual @ huggingface

pgvector (vector storage, ex: pg extension)

[github.com/ankane/neighbor](http://github.com/ankane/neighbor)

github: prologmqi.rb, another prolog interface for Ruby (Ruby binding prolog)

用 prolog 去做 deduction 去提高 propositions

中間有一增 DSL

可以直接將 prolog 的結果送去 gpt 串接

或用 format you define 去檢查 gpt 送回來的結果

或在這層自己去實現你要的檢查方法

demo

security guard game?

the DSL is under developing, maybe release April or May next year.