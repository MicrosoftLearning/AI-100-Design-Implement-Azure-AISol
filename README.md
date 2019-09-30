# AI-100-DesignImplementAzureAISol
Lab files for AI100T01A ILT Course


## Lab 1: Meeting the Technical Requirements

In this lab, we will introduce our workshop case study and setup tools on your local workstation and in your Azure instance to enable you to build tools within the Microsoft Cognitive Services suite.

## Lab 2: Implement Computer Vision Capabilities for a Bot

This hands-on lab guides you through creating an intelligent console application from end-to-end using Cognitive Services (specifically the Computer Vision API). We use the ImageProcessing portable class library (PCL), discussing its contents and how to use it in your own applications.

## Lab 3: Basic Filtering Bot

In this lab, we will be setting up an intelligent bot from end-to-end that can respond to a user's chat window text prompt. We will be building on what we have already learned about building bots within Azure, but adding in a layer of custom logic to give our bot more bespoke functionality.

This bot will be built in the Microsoft Bot Framework. We will evoke the architecture that enables the bot interface to receive and respond with textual messages, and we will build logic that enables our bot to respond to inquiries containing specific text.

We will also be testing our bot in the Bot Emulator, and addressing the middleware that enables us to perform specialized tasks on the message that the bot receives from the user.

We will evoke some concepts pertaining to Azure Search, and Microsoft's Language Understanding Intelligent Service (LUIS), but will not implement them in this lab.

## Lab 4: Log Bot Chat

In the previous lab, we started with an echo bot project and modified the code to suit our needs.  Now, we wish to log chats with our bots to enable our customer service team to follow up to inquiries, determine if the bot is performing in the expected manner, and to analyze customer data.

This hands-on lab guides you through enabling various logging scenarios for your bot solutions.

In the advanced analytics space, there are plenty of uses for storing log conversations. Having a corpus of chat conversations can allow developers to:

1. Build question and answer engines specific to a domain.
2. Determine if a bot is responding in the expected manner.
3. Perform analysis on specific topics or products to identify trends.

In the course of the following labs, we'll walk through how we can enable chat logging and intercept messages. We will evoke some of the various ways we might also store the data, although data solutions are not within the scope of this workshop.

## Lab 5: QnA Maker

In this lab you will use the Microsoft QnA Maker application to create a knowledgebase, publish it and then consume it in your bot.

## Lab 6: Implement the LUIS model

We're going to build an end-to-end scenario that allows you to pull in your own pictures, use Cognitive Services to find objects and people in the images, and obtain a description and tags. We'll later build a Bot Framework bot using LUIS to allow easy, targeted querying.

## Lab 7: Integrate LUIS into a bot with Dialogues

Now that our bot is capable of taking in a user's input and responding based on the user's input, we will give our bot the ability to understand natural language with the LUIS model we built in [Lab 6](/Lab6-Implement_LUIS/02-Implement_LUIS.md)

## Lab 8: Detect User Language

In this lab we will add the ability for your bot to detect languages from user input.

If you have trained your bot or itegreated it with QnA Maker but have only done so using only one particular language, then it makes sense to info users of that fact.  

## Lab 9: Test Bots in DirectLine

This hands-on lab guides you through some of the basics of testing bots. This workshop demonstrates how you can perform functional testing (using Direct Line).
