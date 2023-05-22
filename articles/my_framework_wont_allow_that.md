---
title: My Framework won't allow that
layout: default
---

In a recent engagement with a client's team, we faced an interesting challenge.

After going through some workshops on good design principles (Mostly about [SOLID principles](https://en.wikipedia.org/wiki/SOLID), we reached a consensus around the benefits of using [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) and wrapping external dependencies with proprietary code.

Those were the good news. 

But then, a kind of reality check came to us.

The User Story we were working on in a [mob](https://www.agilealliance.org/glossary/mob-programming/) fashion was supposed to interact with a [Kafka server](https://kafka.apache.org/), so we put together a class called <code>KafkaConnectorService</code>, which looked somewhat like this:

{% highlight javascript %}
export class KafkaConnectorService {
private kafkaProducer: Producer;
public readonly kafkaClient = this._kafkaService.createKafkaClient();

            constructor(private readonly _kafkaService: KafkaService) {
                this.resolveProducer();
            }

            public async sendMessage(msg: string): Promise&lt;void&gt; {
                if (this.getKafkaTopicSC()) {
                    this.kafkaProducer.send({
                        topic: this.getKafkaTopicSC(),
                        messages: [{ value: msg }],
                    });
                }
            }

            private getKafkaTopic() {
                return process.env.KAFKA_TOPIC;
            }
        }
{% endhighlight %}

In case you're not familiar with the syntax, this code is written using [TypeScript](https://www.typescriptlang.org/) but the principles discussed here apply to any other.

You've probably already noticed but, in case you didn't, let me draw your attention to the line <code>return process.env.KAFKA_TOPIC;</code>.

If we were to leave it like that, we would effectively be creating an unneeded coupling between the application and the environment.

What if, down the road, we needed to switch the configuration storage mechanism to something else? 

Too speculative for you?

Fair enough, let's look at it from a more _pragmatic_ point of view: how do we write tests for this code?

If we were to keep this structure, the only way to do some testing would be to save the environment before each test and restore it afterward.

Not too complex but again, that wouldn't complain with testing good practices (Tests should be independent of each other).

A more interesting solution is, precisely to use dependency injection, like this:

{% highlight javascript %}
        export class KafkaConnectorService {
            private kafkaProducer: Producer;
            public readonly kafkaClient = this._kafkaService.createKafkaClient();
            private topic: string;

            constructor(private readonly _kafkaService: KafkaService, topic: string) {
                this._topic = topic;
                this.resolveProducer();
            }

            public async sendMessage(msg: string): Promise&lt;void&gt; {
                if (this.getKafkaTopicSC()) {
                    this.kafkaProducer.send({
                        topic: this.getKafkaTopicSC(),
                        messages: [{ value: msg }],
                    });
                }
            }

            private getKafkaTopic() {
                return this.topic;
            }
        }
{% endhighlight %}

## What is the problem?

The problem the team faced (expressed is probably a more accurate description) was the fact that [NestJs](https://nestjs.com/), the framework being used in the project, was _magically_ creating the instance of the service, so there was no way to inject this dependency from the outside.

When I first heard the team's arguments, I was skeptical, given my experience using other frameworks in other languages [PHP's Symfony in this case](https://symfony.com/) in this case I knew this was possible.

Luckily, after a little digging, we realized this assumption was not true and the framework did allow such initialization, so we were able to complete the expected tests and functionality without breaking good design rules.

## The lesson

Probably many things can be taken from this experience, but I want to focus on two I find particularly important:

1. **A framework is a great tool**. In today's landscape, it's extremely unlikely that you'll be better off without a framework for any reasonably sized project. The alternative is for your team to build everything from scratch, which would be a huge waste of time and resources.
1. **The framework is your tool**. By this I mean, the framework, as any other tool, should work for you (a.k.a. make your life easier), not the other way around. While this may seem obvious, it's sometimes forgotten in the heat of the battle.
1. **If the framework doesn't allow you to do something it should, you probably don't know the framework well enough**. It's very common for developers to believe that since they have extensive experience using a tool they know all there is to know about it. If you find this to be your case, my advice is to give yourself the benefit of the doubt and do a little more research. Most modern and mature frameworks are really powerful tools, created by very smart people who face (or have faced) similar issues as your own so chances are somebody already came up with the solution.
1. **If the framework forces you to ignore best practices, change the framework**.
   Ok, I realize this one can, many times,
   be out of the real possibilities of a team/project but should definitely be considered.
   A tool that forces you to produce mediocre code is not one you want to use (See the first point).

I hope this article gave you a new perspective on your particular situation.
Cheers,