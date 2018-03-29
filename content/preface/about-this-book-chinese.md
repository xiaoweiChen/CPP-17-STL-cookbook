# What you need for this book 

All recipes in this book are kept as simple and self-contained as possible. They are easy to compile and run, but depending on the reader's choice of operating system and compiler, there are differences. Let's have a look how to compile and run all the recipes, and what else to pay attention to. 



# Compiling and running the recipes 

All the code in this book has been developed and tested on Linux and MacOS, using the GNU C++ compiler, **g++**, and the LLVM C++ compiler, **clang++**. 

Building an example in the shell can be done with the following command using g++: 
`$ g++ -std c++lz —o recipe_app recipe_code. cpp `

Using clang++, the command is similar: 
`$ clang++ -std C++ Iz -o recipe_app recipe_code. cpp `

Both the command-line examples assume that the file `recipe_code. cpp` is the text file containing your C++ code. After compiling the program, the executable binary will have the filename `recipe_app` and can be executed as follows: 
`$ . /recipe_app `

In a lot of examples, we read the content of entire files via standard input. In such cases, we use the standard UNIX pipes and the UNIX `cat` command to direct the file content into our app, as follows: 
`$ cat file. txt | ./recipe_app`

This works on Linux and MacOS. In the Microsoft Windows shell, it works as follows: 

`> recipe app. exe < file.txt`

If you do not happen to run your programs from the command-line shell, but from the Microsoft Visual Studio IDE, then you need to open the dialogue, `"Configuration properties > Debugging "`, and add the`"< file. txt"` part to the command line of the app that Visual Studio uses for launching. 



# Requirements for early adopters 

If you happen to read this book in the earliest days of C++17 and use bleeding- edge compilers to compile the code, you might experience that some recipes do not compile yet. This depends on how much of the C++17 STL has been implemented already in your STL distribution. 

While writing this book, it was necessary to add the path prefix `experimental/` to the headers`<execution_policy>` and `<filesystem>`. There might also be additional includes such as algorithm, numeric, and so on, in the `experimental/ `folder of your ST L distribution, depending on how new and stable it is. 

The same applies for the namespace of brand new features. The parts of the library that were included from the experimental part of the STL are usually exported not within the `std` namespace but the `std: : experimental` namespace. 



# Who this book is for 

This book is not for you if you have no prior knowledge of writing and compiling C++ programs. If you read about the basics of this language already, this book is the ideal second book about C++ to take your knowledge to an advanced level. 

Apart from that, you are a good candidate for reading this book if you can identify yourself with one of the following bullet point descriptions: 

You have learned the basics of C++, but now, you don't have a clue where to go next, since the gap between your knowledge and the knowledge of an experienced C++ veteran is still large. You know C++ well, but your knowledge of the ST L is limited. You know C++ from one of the older standards, such as C++98, C++11, or C++14. Depending on how far in the past you used C++ the last time, this book has a lot of nice new STL features and perks in store, ready for you to discover. 



# Sections 

In this book, you will find several headings that appear frequently (Getting ready, How to do it, How it works, There's more, and See also). 

To give clear instructions on how to complete a recipe, we use these sections as follows: 

## Getting ready 

This section tells you what to expect in the recipe, and describes how to set up any software or any preliminary settings required for the recipe. 

## How to do it... 

This section contains the steps required to follow the recipe. 

## How it works... 

This section usually consists of a detailed explanation of what happened in the previous section. 

## There's more... 

This section consists of additional information about the recipe in order to make the reader more 
knowledgeable about the recipe. 



## See also 

This section provides helpful links to other useful information for the recipe. 



# Conventions 

In this book, you will find a number of text styles that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning. 

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "The next step is to edit 
`build. properties ` file." 

A block of code is set as follows: 

```c++
my_wrapper<T1, T2, T3> make wrapper (Tl t 1, T 2 t2, T3 t3) 
{
  return t 1, t2, t3;
}
```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this: "Once done, click on Activate." 
*Warnings or important notes appear in a box like this.*
*Tips and tricks appear like this.*



# Reader feedback 

Feedback from our readers is always welcome. Let us know what you think about this book-what you liked or disliked. Reader feedback is important for us as it helps us develop titles that you will really get the most out of. To send us general feedback, simply e-mail `feedback@packtpub. com`, and mention the book's title in the subject of your message. If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide at www.packtpub.com/authors. 



# Customer support 

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase. 



# Downloading the example code 

You can download the example code files for this book from your account at http://www.packtpub.com. If you purchased this book elsewhere, you can visit http://wmv.packtpub.com/support and register to have the files e-mailed directly to you. 

You can download the code files by following these steps: 

- Log in or register to our website using your e-mail address and password. Hover the mouse pointer on the SUPPORT tab at the top. Click on Code Downloads & Errata. Enter the name of the book in the Search box. Select the book for which you're looking to download the code files. Choose from the drop-down menu where you purchased this book from. Click on Code Download. 
- Once the file is downloaded, please make sure that you unzip or extract the folder using the latest version of: WinRAR / 7-Zip for Windows Zipeg / iZip / UnRarX for Mac 7-Zip / PeaZip for Linux 

The code bundle for the book is also hosted on GitHub at https://github.com/PacktPublishing/Cpp17-STL-Cookbook also have other code bundles from our rich catalog of books and videos available at 
https://github.com/PacktPublishing/. Check them out! 



# Errata 

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a 
mistake in one of our books-maybe a mistake in the text or the code-we would be grateful if you could report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting http://mvw.packtpub.com/submit-errata, selecting your book, clicking on the Errata Submission Form link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website or added to any list of existing errata under the Errata section of that title. 
To view the previously submitted errata, go to https://mwv.packtpub.com/books/content/support and enter the name of the book in the search field. The required information will appear under the Errata section. 



# Piracy 

Piracy of copyrighted material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works in any form on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy. 

Please contact us at `copyright@packtpub. com` with a link to the suspected pirated material. 

We appreciate your help in protecting our authors and our ability to bring you valuable content. 

# Questions 

If you have a problem with any aspect of this book, you can contact us at `questions@packtpub. com`, and we will do our best to address the problem. 