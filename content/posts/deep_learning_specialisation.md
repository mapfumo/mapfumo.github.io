---
title: "Deep Learning Specialisation - A Brief Course Review"
date: 2023-04-21T18:00:58+10:00
draft: false
cover:
    image: /img/deep_learning_certificate_specialisation.png
    alt: 'Antony mapfumo - Deep Learning Certificate'
    caption: 'Deep Learning Certificate'
tags: ["python", "deep learning"]
categories: ["AI", "Tech"]
---

I have just completed [Andrew Ng's](https://www.coursera.org/instructor/andrewng) [Deep Learning Specialisation](https://www.coursera.org/specializations/deep-learning) course by [deeplearning.ai](https://www.deeplearning.ai/) available through [Coursera](http://bit.ly/2WjYrPB). This is my summary and opinion of the course offering. The specialisation consists of 5 courses and it is suggested that they be completed in 3 months by devoting 11 hours per week. It really depends on your previous knowledge, experience and how quickly you can grasp the concepts. When stuck with the assignments and concepts I found the forum to be very helpful. I found the assignments to reasonably difficult. The only thing I didn't like is that by forcing you to complete the given code (complete missing blanks) you are a bit constrained. For example it would be nice to state the function signature and leave it to the student to implement it in their own way. The good thing is that one can always make such suggestions through the forums. The courses don't have to be completed in any particular order but I found it more helpful to follow the suggested order.

#### Course 1 - Neural Networks and Deep Learning

This is the first course in the specialisation. This is a good course if you are just getting started with deep learning. I already had a good foundation of neural networks and deep learning from reading several books and watching YouTube videos. I was however pleasantly surprised by the way Andrew explained the concepts in this course. Here are some of the highlights:-

1. Deep neural networks from the first principles, overview (representation and mathematical concepts), logistic regression
2. Derivatives - the mathematical application of derivatives to deep learning algorithms like *gradient descent* and *backpropagation* are introduced. It is very helpful is you have prior knowledge of calculus however the instructor does well to explain the applicable concepts here.
3. Vectorization - the importance of avoiding "for loops" in your code and appreciation of the speed advantage gained by vectorisation offered by the *numpy* Python library when implementing deep learning algorithms
4. Activation functions

#### Course 2 - Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization

This is one of the shortest (the other being Sequence Models) of the five courses with a 3 week completion time recommendation. I think the concepts in this course are important enough to be covered comprehensively in about 5 weeks.
*Regularisation* as a solution to the problem of over-fitting is explained. In addition to *dropout* and *L2 regularisation* other techniques like *data augmentation* (increasing data diversity by cropping and padding for example) *early stopping* are explained. Solutions to the vanishing gradient problem like choosing reasonable scaling when initialising the weights. Optimisation techniques include mini-batch gradient descent. Ability to  implement and apply a variety of other optimisation algorithms as *Momentum, RMSprop* and *Adam*, and check  for their convergence. Implementing neural networks in TensorFlow version 1.x is introduced. I would suggest introducing version 2.0 as it has been out for a while.

#### Course 3 - Structuring Machine Learning Projects

Of the 5 courses this is the least technical. Here are some takeaways from the course:-

- ML algorithms can compete with human-level performance since they are more productive and more feasible in a lot of application.
- Building an algorithm: the progress tends to be relatively rapid as you approach human level performance. When the algorithm surpasses human-level performance the progress and accuracy actually slows down.
- Bayes optimal error (lowest possible prediction error that can be achieved and is the same as irreducible error) is the best possible error.
- Human level error is worse than Bayes error because nothing could be better than Bayes error but human level error might not be too far from Bayes error.
- Avoidable bias: The difference between the training set error and the human level error.
- Bias reduction techniques to reduce the variance between the training set and the human level error: training a bigger neural network or running the training set longer.
- Variance reduction techniques to reduce the variance between the training error and the development error: regularisation or have a bigger training set.
- Making progress in a machine learning problem gets harder as you achieve or as you approach human-level performance.
- In real-life projects, there is no particular expectation to get 0% error. Because sometimes Bayes error is non zero and sometimes it’s just not possible for anything to do better than a certain threshold of error.
- Having an estimate of human-level performance gives you an estimate of Bayes error. And this allows you to more quickly make decisions as to whether you should focus on trying to reduce a bias or trying to reduce the variance of your algorithm.
- Problems where ML significantly surpasses human-level performance: online advertising, product recommendations, logistics (predicting transit time) and loan approvals.
- The machine learning strategy is how to choose the right direction of the most promising things to try.

#### Course 4 - Convolutional Neural Networks

This was the most enjoyable (maybe because I am interested in computer vision) of the 5 courses as the explanations and examples were very clear. I enjoyed the [YOLO](https://pjreddie.com/darknet/yolo/) object detection algorithm section. This course will teach you how to build convolutional neural networks and apply it to image data. The main sections are:-

- Foundations of Convolutional Neural Networks
- Deep convolutional models: case studies
- Object detection
- Bias reduction techniques to reduce the variance between the training set and the human level error: training a bigger neural network or running the training set longer.
- Variance reduction techniques to reduce the variance between the training error and the development error: regularisation or have a bigger training set.
- Making progress in a machine learning problem gets harder as you achieve or as you approach human-level performance.
- In real-life projects, there is no particular expectation to get 0% error. Because sometimes Bayes error is non zero and sometimes it’s just not possible for anything to do better than a certain threshold of error.
- Having an estimate of human-level performance gives you an estimate of Bayes error. And this allows you to more quickly make decisions as to whether you should focus on trying to reduce a bias or trying to reduce the variance of your algorithm.
- Problems where ML significantly surpasses human-level performance: online advertising, product recommendations, logistics (predicting transit time) and loan approvals.
- The machine learning strategy is how to choose the right direction of the most promising things to try.

#### Course 4 - Convolutional Neural Networks

This was the most enjoyable (maybe because I am interested in computer vision) of the 5 courses as the explanations and examples were very clear. I enjoyed the [YOLO](https://pjreddie.com/darknet/yolo/) object detection algorithm section. This course will teach you how to build convolutional neural networks and apply it to image data. The main sections are:-

- Foundations of Convolutional Neural Networks
- Deep convolutional models: case studies
- Object detection
- Special applications: Face recognition & Neural style transfer

#### Course 5 - Sequence Models

In this course you learn about recurrent neural networks, RNNs, and its LSTMs, GRUs and Bidirectional RNNs variants.

Using word vector representations and embedding layers you can train recurrent neural networks. Example applications are sentiment analysis, named entity recognition and machine translation.

#### Conclusion

Which course stood out? For me it was course 4, Convolutional Neural Networks. This is one of the best explanations of CNNs I have come across. This whole specialisation course covered enough theoretical ground for me to move on to more practical courses. I am now equally comfortable in applying the concepts to both the TensorFlow and PyTorch deep learning frameworks.

You get a certificate after successfully completing each course in the specialisation and a final [verified deep learning certificate](https://www.coursera.org/account/accomplishments/specialization/PCWQGFCUM5BN) upon finishing all 5 courses.
