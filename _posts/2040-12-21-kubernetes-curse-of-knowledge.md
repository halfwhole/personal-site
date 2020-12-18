---
layout: post
title:  "Kubernetes and the Curse of Knowledge"
date:   2040-12-21 09:00:00 +0800
tags:   kubernetes
---

<This is so not done>

I've been using Kubernetes recently as a cloud systems intern, and one thing that frustrated me was just how difficult it was to learn.

I'm not at all here to disparage Kubernetes. It's great at managing and coordinating large-scale applications, and much of its complexity is certainly unavoidable. Rather, my complaint with Kubernetes lies with its documentation: Kubernetes may be hard, but its bad documentation only makes it even harder.

| ![What is Kubernetes?](/public/images/kubernetes/what-is-kubernetes.png) |
|:--:|
| *Thanks, I sure needed to read that paragraph twice.* |

To be fair, the [Hello Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) tutorial serves as a great 5-minute introduction---it's simple to follow, and briefly introduces some of Kubernetes' key concepts, such as Pods, Deployments, and Services. But from then on, you're on your own. To learn more about these ideas, you'd be forced to turn to their [Concepts](https://kubernetes.io/docs/concepts/overview/) section, which is as comprehensive as it is intimidating. Each page comes with a mass of technical terms and unnecessary information, confusing the poor reader who lacks the fundamental knowledge to understand it.

Introduce unnecessary complexity, information overload and 'dump'

I have a feeling that it would be better explained if they didn't pound the readers with all the information at once, and thought carefully about best how to introduce new concepts, and created guides with examples in tandem with exploring these concepts, instead of 'dumping'/'unloading' all they know onto the page.
- Take a look at Istio's page or Docker's page: new concepts are always brought in with examples to illustrate the point
- As a reader, I felt challenged, but never lost

Whoever wrote this failed to see it from the point of view of the user, who knows next-to-nothing

## The Curse of Knowledge

This failure is a symptom of the 'curse of knowledge'.

I suspect many developers suffer from it, in communicating their work to those who have never used it before


My experiences with learning other technologies, such as Docker and Vagrant, were far more pleasant. These tools are less complex, to be sure, but its guides were excellent and its examples were well-placed. To the authors, thank you! :)

Why developers seem so quick to shun documentation, when it is so important to one's understanding.

More generally, I believe that documentation should _never_ be an afterthought. Development is never only a technical endeavour, but a social one too: after all, we write things for people to read, and only incidentally for machines to execute.
