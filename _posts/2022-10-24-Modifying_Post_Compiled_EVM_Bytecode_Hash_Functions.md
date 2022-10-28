---
title: Modifying Post Compiled EVM Bytecode Hash Functions
date: 2022-10-24 8:00:00 +/-1111
author: Jacob Vaughn
categories: [Posts]
tags: [evm, bytecode, function, hashing, posts]
---

## Overview:
As you may or may not know hashed functions are the method by which interaction with EVM smart contracts is possible. Hash functions are exactly what they sound like, functions such as `` or `` that are hashed with "" into 4 byte during compile time resulting in hex code such as `` or `` for the respective previous functions. If you are interested in more depth there is a great article `here` that goes further. 
These functions are important to understanding what the functions name is and what parameters are needed in order for it to provide is well... function. In this post I will go over how to change these function hashes of post compiled smart contracts and some of the reasons someone might want to do this, or how it could potentially be used for nefarious not so pure intentions.

## How I Happened Upon Finding This
So I believe this post will make a bit more sense if I give the context as to how I came to understanding this was possible. If you do not care or wish to know the answer to how it is done skip down to the "How to Modify Function Hashes" section.

As we all may know there are some incredibly smart people out there especially in the blockchain space (maybe you? 0_o) and these guys or gals have a big leg up when it comes to smart contract design and efficiency. Safe to say I am not there yet so I need to get a little creative in order to compete. To do this I thought hey if all bytecode is available on the network why don't I just use already deployed smart contracts for my own purposes. Now I know this might not be the most wholesome or safe idea and if anyone elses funds were at risk I would not be doing it, but for me it is a great learning experience. The specifics of how I am able to use smart contracts with only compiled bytecode, not having the source code or knowing how it works is not for this post, but I may make one some other time.

Back to the main topic, because I am using unknown smart contracts it opens me up to a lot of vulnerabilities as I do now fully know what the smart contract code I am using is capable of. To mitigate this risk I wanted to try and make my use of the smart contract code less obvious to the original creator or creators. At first I though the best way would be to create another smart contract that would then call my first contract changing the function hashes used and making it difficult to detect on tools like etherscan. I quickly remembered while doing this that smart contract addresses are `!=` to normal user signing addresses and therefor functions which use `require _param1 == addr(_param1)` revert instantly. Back to the scratch board.

I then though "Hey hashed functions shouldn't care what their original names or parameters were hashes are usually a one way operation. I should be able to in theory change them to anything I wanted and then call them with the new functionHash." After thinking this I did a little testing and found out "Hey it works!... well sort of?" I found that some hex codes would work and others wouldn't, but I wasn't entirely sure why. See at the time I was only using `panoramix` to decompile the smart contracts, so I did not understand that the compiler does a small optimization for function calls. Upon using another decompiler `put compiler here` I realized that the compiler finds the function hash that is numerically between the others and uses a simple `if <=` to divide the search time in half. 

An example of what I mean is below in the screenshot from ethervm.io, and if you are still confused I will explain more in the detailed section below. After I understood this I was able to change my function hashes deploy my smart contracts and hopefully fingers crossed to undetected, until I learn enough to make my own smart contracts. ^_^



## How to Modify Function Hashes
First you are going to need the bytecode of your target contract that was used in the smart contract creation. There are two types of bytecode usually named bytecode and deployed bytecode if you do not know the difference I suggest reading this medium post https://medium.com/coinmonks/the-difference-between-bytecode-and-deployed-bytecode-64594db723df. Next we will take our bytecode and put it into https://ethervm.io/decompile to obtain the function hashes. For this example we are using a simple deposit / withdraw contract.

BYTECODE
`INSERT BYTECODE`

Decompiled Contract by ethervm.io
`INSERT DECOMPILED SMART CONTRACT HERE`

Now that we have our functions we can identify them in the hex compiled smartcontract code aka the bytecode.

`Show functions Highlighted in the bytecode`

We can then change the 4 bytes to any desired 4 bytes we want with one big exception. If you already figured it out the new function we pick must be smaller then or greater than the numerical mid point function depending on if the original function was smaller or greater. For example in our smart contract above `0xTODO` is the midpoint function and if we want to replace `0xTodo` we would need to pick a hex number smaller than `0xTODO`. Or if we want to change `0xTODO` we would need to change it to a hex number larger than `0xTODO`. This can be circumvented by also changing the mid point function, but keep in mind that because they bytecode has already compiled it will not change the order nor which functions are greater than or less than the mid point function. For example if I change the mid point to the largest number possible `0xffffffff` then I can choose any new hash function for `0xTODO` and `0xTODO`. But I will not be able to use functions `0xTODO` and `0xTODO` because there is no function I can choose that is greater than `0xffffffff`. It should also be noted that changing two or more hashes to the same hash results in unique scenarios, but may not be desireable. 

Now that we have changed one or all of the function hashes keeping in mind the exceptions above, we can deploy the smart contract using any tool that allows us to send a transaction with the data field. My favorites for quick deployment are ethersJS with hardhat and https://lovethewired.github.io/abi-playground. All you need to do then is to wait for your transaction to be mined and use your contract with the new function hashes. I currently call functions by creating my own transaction data but there might be a better more user friendly way using hardhat, but for now it works.

## Real World Implications
Okay now for the real question. Why would anyone want to do this? Well if you didn't read my "How I Happened Upon Finding This" section then I believe the main use case would be to obfuscate what your smart contract functions are and what they do. There are databases with well known functions such as https://www.4byte.directory/ which has a large number of hash functions for look up sourced from verified code and github projects. These databases make it easy to see what functions are used by a smart contract and enable you to interact with them with out the source code.

This is where we get to the good stuff and the potentially unintended or intended malicious use of this knowledge that I believe security minded individuals should be aware of. Because we can change the function hash to any other that we choose, there is nothing stopping us from changing them to common functions found in the databases. This could be a problem if you were to trust a contract purely based on the function name showing up in a service like etherscan. Etherscan will only look at they function hash in the data field of a transaction and assign the name of the function without the contract being verified. This is not too big of an issue as I don't believe anyone actually does this, and there are other ways of doing this like creating a function with the same name and parameters in source code. Anyone aware that this is possible shouldn't be susceptible to this trick, but to those unaware they should keep it in mind.

The last reason I can come up with that anyone would want to do this is simply for fun. You can make any function look like another. I could have a NFT minter saying that it is swapping. I could have a function that looks like it only has one uint256 parameter, but actually needs 7 different variables uint8, bytes, account arrays, ect. The only other way I can see changing the number of parameters of a function without changing the hash or name is by hashing a bunch of functions with different names to get a collision with the target function. 

## Conclusion
I hope you enjoyed this novel look into post compiled evm hex hash functions. I could not find any other post about this subject manner on the internet so I decided to post this. Feel free to message me if someone else has already detailed this, you have any questions, or want to give me feedback. Thanks for reading!

P.S. I recommend giving this a try out yourself with tools such as etherJs with hardhat or https://lovethewired.github.io/abi-playground and see how fun it is.
