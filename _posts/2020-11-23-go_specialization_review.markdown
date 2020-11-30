---
layout: post
title: Programming with Google Go Specialization - A Brief Course Review
date: 2020-11-23 19:55:12
categories: software-development
tags: [software-development, courses]
typora-root-url: ../img
---

Go or [GoLang](golang.org) is an open source statically typed language that was created at Google by Rob Pike, Robert Griesemer, and Ken Thompson. It first appeared in Nov 2009 and has been rapidly gaining in popularity. Some of the language's highlights include clean and highly accessible syntax, garbage collection,  amazing native concurrency, fast compilation speed, excellent tooling, builtin documentation, good cross-platform support, ORM (Object-relational mapping ) library called GORM and excellent support for micro-services. 

I have just completed the ["Programming with Google Go Specialization"](https://www.coursera.org/specializations/google-golang?) course by University of California, Irvine available through [Coursera](http://bit.ly/2WjYrPB) and Instructed by [Professor Ian Harris](https://www.coursera.org/instructor/ianharris).  This is my summary and opinion of the course offering. The specialisation consists of 3 courses (Getting Started with Go, Functions, Methods, and Interfaces in Go, Concurrency in Go) and it is suggested that they be completed in 3 months by devoting 2 hours per week.



#### Course 1 - Getting Started with Go



This is the first course in the specialisation. Here you are introduced to the specialization. You will  learn the basics like data types (strings, integers, floating points, etc) and complex types like maps, structs and slices. If you have used other languages like Java, C/C++ or Python the syntax and concepts and applied to Go will be easy to grasp.

#### Course 2 - Functions, Methods, and Interfaces in Go

This is one of the shortest (the other being Sequence Models) of the five courses with a 3 week completion time recommendation. I think the concepts in this course are important enough to be covered comprehensively in about 5 weeks. 

*Regularisation* as a solution to the problem of over-fitting is explained. In addition to *dropout* and *L2 regularisation* other techniques like *data augmentation* (increasing data diversity by cropping and padding for example) *early stopping* are explained. Solutions to the vanishing gradient problem like choosing reasonable scaling when initialising the weights. Optimisation techniques include mini-batch gradient descent. Ability to  implement and apply a variety of other optimisation algorithms as *Momentum, RMSprop* and *Adam*, and check  for their convergence. Implementing neural networks in TensorFlow version 1.x is introduced. I would suggest introducing version 2.0 as it has been out for a while.

#### Course 3 - Concurrency in Go

Concurrency can be difficult in other programming languages. Unlike other languages that use threads to implement concurrency Golang makes concurrency tractable and pleasant to implement using cheap and easy goroutines (lightweight, user-space threads that are managed by the Go runtime). 

In this course the need for concurrency in relation to microprocessor performance is discussed. Basic concurrency concepts and race conditions followed by threads in Golang are introduced. Implementing channels for communication between goroutines is demonstrated



An interesting demonstration of concurrent algorithms and synchronisation issues is the final classic ["Dining Philosopher"]() https://en.wikipedia.org/wiki/Dining_philosophers_problem) programming assignment.

#### Conclusion

This is an intermediate Golang course and sets the a foundation for exploring more advanced topics. The forums are helpful for general discussion and assignments should you get stuck.

You get a certificate after successfully completing each course in the specialisation and a final [verified deep learning certificate](https://www.coursera.org/account/accomplishments/specialization/KKHD53ZLEVNL) upon finishing all 3 courses. ![](/img/go_programming_certificate_specialisation.png)

