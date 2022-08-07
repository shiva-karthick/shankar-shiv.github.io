---
layout: post
title:  "Singapore Armed Forces Digital Checklist Application"
date:   2022-07-25 13:00:00 +0800
categories: app-development,flutter,dart
mathjax: true
---


Today marks one year since my ORD (Operationally Ready Date), at which point I began the ORNS (Operationally Ready National Service) phase. I was remembering the good times I had in my unit and throughout BMT (Basic Military Training). When I was in my Maintenance Unit, I developed a TIC (Task Instruction Checklist) mobile application. I'll go into greater detail about the technical aspects and my learning experience from developing a mobile application based on the Flutter framework.

# Purpose

My Maintenance Unit has always used paper and pen in the conventional manner to annotate completed tasks, pending tasks, faulty and missing equipments. As part of [SAF's digitalization efforts](https://www.channelnewsasia.com/singapore/mindef-army-camps-integrated-smart-technologies-saf-digital-transformation-efforts-2783076), I have simplified the process of collecting and consolidating data, analysing the results and finally synchronising the data to Maintenance Officers (MTT). 

In other words, it's a *digital* version of the paper checklist with intelligent features like data analysis and accurate data entry, which reduces the amount of information that is duplicated.

# User Diagram

![integrated_change_control](/assets/images/TIC-Checklist-Application/user_diagram.png)

This simple architecture as shown in the image above, depicts the intended way of using the application. The data files are stored and accessed in JSON format and are encrypted before sending and decrypted after receiving from the other person.

# The Design Phase

![Initial_Form_Sketch](/assets/images/TIC-Checklist-Application/Initial_Form_Sketch.jpg)

I went through numerous iterations of creating design and functions of the user interface, every time seeking to improve user's experience while using the app. One of such prototype is shown in the image above. There were many design decisions to be considered such as the arrangement of the missing/faulty equipments, how much padding to add in the text boxes and the layout of the form elements. I found this to be a difficult challenge because I had no prior design experience. I devised a strategy to make many prototypes and ask my friends to use the application without my assistance. Because I was able to identify my design flaws and receive rapid feedback, this strategy proved to be beneficial. The final form design was therefore acceptable.

<img src="/assets/images/TIC-Checklist-Application/Final_Form_Design.jpg" alt="brainstorming" style="width:376px;height:820px;"> 

# The Development Phase aka horror phase 😓

I have little experience with the hybrid approach to developing mobile apps. Because there are so many frameworks available for developing apps, like Flutter, React Native, Xamarin, PhoneGap, and others, I was rather perplexed. Eventually, I settled down to Flutter vs React Native. To decide which framework to adopt, I was googling for opinions from experienced developers in reddit and stack overflow. I was still not sure which framework works best for my application. 

#### Flutter

I quite liked the Skia, the graphics engine used by Flutter, which is an open source 2D graphics library responsible for the buttery smooth animations, rendering better performance and providing an eye candy for users. Additionally, Flutter's just in time (JIT) compilation speeds up code updates reducing development time. JIT is also responsible for the hot reload capabilities of Flutter. I rather enjoyed the idea of using widgets modeled after React, which are used to  design user interface (UI), provide descriptions of how views should appear given their configuration and current state. When a widget's state changes, it rebuilds its description. The framework then compares this description to the previous description to figure out what minimal changes are required in the underlying render tree to get from one state to the next. To gauge my proficiency with the Dart language and the Flutter environment, I created a straightforward calculator application. It was a wonderful to create applications using Flutter, and the development process was amazing satisfying.

#### React Native

Due to my familiarity with JavaScript, React Native, on the other hand, seemed to be lot simpler to master. On the other hand, Flutter's tooling has a lot more polish than React Native's tooling. When I created the same calculator application in React Native, I faced the horror of dependency management from the *gazillion* amount of packages. Thankfully, yarn resolved the dependency issues, but the incident still terrified me. The third-party libraries support was a key determining factor in my search for the necessary libraries I may need for my application. Since, both frameworks were matured already, there were plentiful libraries available. 

As a result, I squandered two solid months looking for the ideal framework. This was a huge mistake on my part, as valuable development time was lost unnecessarily.

## Model Architecture

![mvvm_general](/assets/images/TIC-Checklist-Application/mvvm_general.png)

I looked at the overall architecture that many Flutter projects employed, as seen in the image above. 
MVVM is useful to move business logic from view to ViewModel and Model. ViewModel serves as a bridge between the view and the model, transferring all user events and returning the outcome. The main advantage is the efficiency you obtain from having a complete separation between the View and Model. In practice, it implies that when your model has to change, it can be altered quickly without requiring a change to the view, and vice versa.

I created a few side projects using the Model-View-View Model (MVVM) to become familiar with the framework and learn how to adapt it for my application.

![mvvm_general](/assets/images/TIC-Checklist-Application/project_architecture.png)

After several attempts, I built my application using the architecture depicted above. Only one service manages the Home view. The Post Models do not require a service which is basically the backend of the application.


## Machine Learning ^_^

In order to identify the equipment that is most likely to malfunction or require maintenance, I added an xGBoost Classification model. As of now only 2 features were considered, the frequency of the item being missing and faulty. The opportunity to build a random forest algorithm from scratch was the most thrilling aspect of this application development. It was at this point where I truly learned to write recursive code. Since decision trees were highly sensitive to small changes of data, I created a tree ensemble for better accuracy.

By the time I was finished, my Depot Commander and Depot Sergeant Major had expressed their appreciation with my work, and as a result, I had received a testimonial of achievements and a plaque honoring my efforts to the 6AMB Unit and 6th Division. This was such a wonderful opportunity for myself to learn while helping to enhance administrative processes and freeing up more time for training.

<img src="/assets/images/TIC-Checklist-Application/TIC.gif" alt="brainstorming" style="width:376px;height:820px;"> 

# Project Timeline

### UX - 23 May 2021

| Task                      | Completed by | Assigned to   | Current Status | Finished |
| ------------------------- | ------------ | ------------- | -------------- | -------- |
| Forms Design              | 23  May      | @shankar-shiv |                | ✅        |
| Card Design in homeScreen | 23  May      | @shankar-shiv |                | ✅        |
| Login Screen              | 23 May       | @shankar-shiv |                | ✅        |
| Charts                    | 25 May       | @shankar-shiv |                | ✅        |
| Undo functionality        | 31 May       | @shankar-shiv |                | ✅        |

### Navigation and Routing

| Task                           | Time required | Assigned to   | Current Status | Finished |
| ------------------------------ | ------------- | ------------- | -------------- | -------- |
| Routing different forms        | 25 May        | @shankar-shiv |                | ✅        |
| Routing from login to homepage | 25 May        | @shankar-shiv |                | ✅        |


### Data manipulation and Backend processes

| Task                            | Time required | Assigned to   | Current Status | Finished |
| ------------------------------- | ------------- | ------------- | -------------- | -------- |
| Capturing data from forms       | 31 May        | @shankar-shiv |                | ✅        |
| Saving JSON files to disk       | 31 May        | @shankar-shiv |                | ✅        |
| Retrieving JSON files from disk | 31 May        | @shankar-shiv |                | ✅        |


That's all folks, appreciate your time reading my blog. Do checkout some of my other posts such as [Solving Markov Chains](http://sshankar.xyz/2022/Solving-Markov-Chains/). Hope you find it useful!