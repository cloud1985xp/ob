---
tags:
  - ruby
  - rails
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Ruby Conf 2023

Created: 2023年12月15日 上午11:33

### A Rails performance guidebook: from 0 to 1B requests/day
Cristian Planas

author worked in Zendesk

Shopify served 79M RPM at peak

ActiveScale? (X)

Scaling

size: 20B tickets, > 200TB RDB

monitoring / error budget

DB Query performance

- DB index
- Sharding
- Greedy select (x)
- Complex DB query, Pareto Principle, the “vital few”, 80-20
- Rapid dataset size
    - IO intensive down
    - Compute intensive upscale
- Denormalize Data writing cache against to, DB
- Trade off
    - Adde complex
    - Cache is not trivial (doraemon pandabox), misuse cache

Cold Storage

- another storage, storage specialized in data that is never/rarely/unfrequently accessed
- mark data as ready for archived
- trade off: multi storage /source cause issus on production, change acess pattern maybe necessary

Design product around performance

- Once platform gets certain size, you no longer what users do
- Growthing payload returned: Sideloads, graphQL
    - Read: Divide data fetch in multi request so users have function
    - Write: Async way? Distributed system : (Eventual) consistency can be painful

Conclusion

- Mindset
- BC333 history story? Alexander …
- tradeoff, no system perfect
- An exclusive pursuit of perfection can be counterproductive

Q&A: When to start considering scaling/performance, like sharding? coldstorage? query performance?

ColdStorage, what kind of coldstorage, what’s the point to consider use different type storage, like dynamo? or even offline database?

## Let's pop into Passkeys

gem “WebAuthn”

github.com/cedarcode/

What are passkeys

Passkeys, replacement for passwords

web auth standard

Passkey is a pub/priv key pair proected by your device biometrics used for chanllenge based authentication

public key cryptograpthy

a secret stored on one’s devices, unlocked with biometrics

Part of webauthn standard

created W3C and FIDO

Public and private key pair

To use it, your device will first execute a biometrics verification

User is asked to sign with private key

Web app/site checks with users’ public key

How it works

Relying Party(your app) → User ask to signup,

- Generate webauthn user id
- load your app web authn settings
- create a challenage
- return a json back to user/browser (specific json for challenge)
    - base on your ruby app settings, like timeout, keyCredParams(ex: algorithm number), extensions, authenticatorSelection
- created for this user’s session, the challenage is used

Ask user to send public key to the app(server) 

(store challenge in session?)

- App response the challenage and user use the challenage to create passkeys

User create a passkeys (with face id & create passkeys) (and sync to cloud account provider)

Got passkeys and send data(with public key and raw data about challenage) back to app(server)

Server

- Verify the data with the challenge from the first step
    - valid type? valid id? , verify data
- Creating user record
    - Store
        - external id (webauthn credential raw_id)
        - public key
        - sign_count (provide by webauth credential)
- create its passkeys
- Return a success response

Authenticate (sign in)

Astk for sign in

Use app to sign in (with biometrics and get passkeys)

create signature with private key

→send signature to server

server compare digital signature, is valid. you are authenticated

Github

- webauthn-with-devise
- ruby-passkeys

### Understanding Parser Generators surronding Ruby with Contributing Lrama

github.com@junk0612

Components of Parsing

- Lexer: program that splits text into tokens (Tokenization) (context information)
- Parser: program that constructs a structure from token stream, ex:
    - Compiler: source code → AST
    - JSON or CSV parsers: text → data structure
- Parser generator: program that generates a parser from grammar(for specific language) files

.rb file → Lexer ⇒ Token Stream → Parser ⇒ AST→ Generator ⇒ Byte codes → Ruby VM

Terms of Programming

- Formal Language
    - the filed of linguistics that deals with “Language” in a mathematical and set-theoretical way
    - Consider how a language is represented as text
        - Does not consider the sematics
        - e.g., English is represented as sequences of alphabets, interspersed with comma
- Context Free Grammar (CFG)
    - A kind of formal language that is presented as follows:
        - rule: A B C …. | DEF …
        - This notation is called Backus-Naur Form (BNF)
    - Almost Programming Languages belongs to this category
    - Used in the grammar file wihich in input
    - Nonterminal Symbol
    - Terminal Symbol

CFG and BNF

BNF is one of the representation of CFG

Contribution in Lrama

github.com/ruby/lrama

Allows for the implementation of ruby-specific features

parsing unfinished code for LSP

making the complex parse.y

Other parse tools surrounding Ruby

- Prism (YARP)
- Bison
- Racc
    - Used in Parser gem (Rubocop dependency)

Named Reference in Bison

Lrama → to Replace Bison

Implement Named Reference in Lrama

Who knows parsing context

Lexer?

Parser?

fromal language

context free gramar

backus -naur form

## Building LLM applications in Ruby

Andrei Bondarev, vedio talk

github: langchainrb

前面在講 AI Train how important until 2030/2060

### Why (not) Ruby

Monoliths are back in fashion

Progmatic community

- Good graps of sound

OOP / Good software dev fundamentals

Ruby ~ Python

similar to python, built lbrary warpped C functions

### What is Generative AI

LLM

Structuing Data: collect and converting unstructrued data, 

Summarizing Data, 

Classifyng Data

problems: 

- hallucinations,
- outdated data,
- relevant knowledge is not used

### RAG

Tech for enhancing the accuracy and reliability of gnerative AI modesl

- vector embedding
- retrieve relevant document related
- send to LLM go get answer back

### Vector embedding

tech to represent data in an N-dimensional space

LLM encode meaning behind tests in the embedding space or ‘latent space’

~ “vector search” or “semantic search” serach by “meaning” (not keyword search)

Similarity search

RAG Prompt

Tip:

- instruction s to enforce a format or style of response
- 

prompt file ⇒ template (.yml)

LLM sythensize the answer

Optimizing RAG

- Human eval (upvote or downvote +1 or -1)
- RAGAS metrics
    - Faithfulness: ensuring retrieved context can acts as a justification for the generated answer
    - Context Relevance: context is focused, with little to no irrelevant information
    - Answer Relevance: the answer addresses the actual question

### AI Agent

- Autonomous (semi-autonomous) general purpose LLM-powered programs
- Can use Tools(APs, other systems)
- Work best with powerful LLMs
- Can be used to automate workflows/business processes and execute multi-step tasks

# Handling 225k requests per second to [RubyGems.org](http://rubygems.org/)

Samuel Giddins

maintainer of rubygems bundler,rubygems.org

security engineer in residence ruby central, responsed by AWS

no revenus on [rubygems.org](http://rubygems.org) :joy

oct 2023

181k users

192k gems

1.5m versions

147.3b total gem downloads

20k req / s

2b req /weekday

max 225k rps

7.5tb/hour, 185 tb/day traffic

how much does it cost

AWS EKS : $20k /month

Fastly, cache, CDN, ddos protect: $500k / month

in free?

Who runs all this?

volunteers and Ruby Central

until 3 weeks ago, nobody worked full time on [rubygems.org](http://rubygems.org) :joy

24/7 on call rotation

only get paged a few times a year for really minor outages

how? to handle this kind of traffic? secret weapon?

don’t serve most of that traffic ourselfs!? (cheat)

Good artist copy

Greate sartis steal,

And smart engineer don no work!

The fastest work you can do is no work at all

Someone else doing the work (is almost as good as no one doing it)?

Rails is amazing, having [rubygems.org](http://rubygems.org) be a rails app written in ruby

but there are limits to what a full-featured web app can serve

We rely havily on letting other simpler and more optimized systems to serve it

Request lifecycle

Each layer is optimized for something different

- Client: (X)
- Fastly edge POP: cache, close to client
- Fastly shield POP (in Seatle): cache, all request to backend flow through a single POP
- Backend, go to S3 or app
    - S3: static content store
    - Rails app
        - AWS ALB
        - nginx: buffering, rules for rate-limiting
        - puma: serves the rails app

### Story Time

aka the weekend sam got paged (fun?), we hit 255k rps

Background:

An (deprecagted) depedncy API lead to 25x increased req.

removing the dependency API

how does it come?

AWS?

New bundler / RubyGems verson mostly stayed the same

Old(really really old) bundler versions fell back to the full index

- Which involves downloading https//rubygems.org/specs.4.8.gz
- Also [indesx.rubygems.org](http://indesx.rubygems.org)/quick/Marshal.xxxx.rails..gemsepec.rz

Misbehaving clients, 10k rps from one IP

We blocked req from that user agent/ip combination

we politely asked that they contact us so we could help them stop DOS

More traffic to https://index.rubygems.orgs/versions

And also more traffic info/rails

Detect: cache missed rate is low(?)

only very rare cache missed

cache misses low(or high) arn’t  normal

large number of cache missed, even 0.1% times  ?

Solution? don’t serve those requests?

Sol: update code

Enqueue a background job on gem push

pre calulate respose

upload response to s3

have fastly serve those request from s3

deploy

Lessons:

- don’ts serve traffic :joy
- if you have to serve traffice, let someone else do it
- prlease don’t DDos yourself (don’t help user DDos your self)
- Find appropiate place to do diff kinds of work, ex CDN, S3
- do all the “normal” optimization work
- use rails caching
- Add indces to your db tables
- set cache-control headers
- sacle up ur DB/web servers
- Add rate limiting
- Be very grateful for sponser who provide those services for free
- Use true but shocking & misleading talk titles :joy
- Ask very politely for people to not install gems when I have brunch plans :joy

不過這裡因為不清楚 gem install 的細節，會怎麼 call api，how 導致大量的 req

所以也無從了解它怎麼做 pre-calculation 預先放到 s3 的

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

1) performance tricks

Ticketed Request (Asynchorouns process)

2) Embedding 

text2vec-base-multilingual @ huggingface

pgvector (vector storage, ex: pg extension)

github.com/ankane/neighbor

github: prologmqi.rb, another prolog interface for Ruby (Ruby binding prolog)

用 prolog 去做 deduction 去提高 propositions

中間有一增 DSL

可以直接將 prolog 的結果送去 gpt 串接

或用 format you define 去檢查 gpt 送回來的結果

或在這層自己去實現你要的檢查方法

demo

security guard game?

the DSL is under developing, maybe release April or May next year.

# Solving Real-World Challenges with Ruby Ractor

Hiếu Nguyễn

Parallelism , Concurrency

GIL

Thread and GIL, shared memory

threads = [10.times.map](http://10.times.map) T[hread.new](http://thread.new) { array << i }

different order but keep always 10 elements

still need to go with limit by GIL

how about ractor

no shared GIL among ractor

Ractor 1 → ractor 1 memory

Ractor 2 → ractor 2 memory

Between ractors: send message

ex: Main Ractor send message to Ractor1 

## Ractor

- Isolated: separated memory
- Message-based communication
- Lightweight

To simplify concurrency, Improve performance

Drawbacks:

- Experiemental
- Unfamiliar interface, lack of libraries support

Example / Use Case

Database encryption, user policy, but need to search (partially) with user information, ex email

Blind index?

input → process → n-gram -hash → token → match? blind index

the process is CPU intensive actions

existing solution

Bank → enroll users in batch → enquueud job (Sidekiq) → workers

Under-utilize CPU , overhead

BlindIndexWorker.perform_later

Apply Ractor

each job is processed in a ractor

- No mutable actions
- avoid blocking operations
- Data is sent via message

[`users.map](http://users.map) do |user|`

  `ractor = Ractor.new(user) do |user|`

      `tokens = user.to_h.slice(required_fields).flat_map { BlindIndexToken.generate(column, value)`

}

  `end`

    [user.id, ractor]

`end`.to_h

ractors.each } do id, ractor|

    tokens = ractor.take

next if tokens.empty?

    user_repo.update(id, blind_index_tokens: tokens)

end

Problem / Learns

- Mutable constant is not shareable
- Method with global state cannot be called

Improvement

- Handle error and retry: single one ractor failed can break whole process
- Get response faster with ractor.select
- Save progress

Conclusion

- Reactor is greate to achieve parallelism in Ruby
- Ractor interface is easy to use and can already replace Thread simple use case
- Ractor is still unstable and more tooling is necessary to support complex use case