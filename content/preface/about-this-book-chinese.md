# 关于本书 

本书中所有的例子都很简单，都可以很容易编译和运行，不过读者们还是需要注意一下自己所选择的操作系统和编译器。下面就让我们来看一下在编译和运行本书例程时，所要注意的一些内容。

# 编译和运行例程 

本书的所有例子都在Linux和Mac OS进行开发和验证，我们使用GNU的C++编译器**g++**, 和LLVM的C++编译器 **clang++**。

 在shell环境下可以使用如下的命令使用g++编译例程：

`$ g++ -std c++lz —o recipe_app recipe_code.cpp `

要使用clang++的话，命令行类似: 

`$ clang++ -std C++ Iz -o recipe_app recipe_code.cpp `

上面两个例子都假设我们的C++例程写在 `recipe_code.cpp`文件中。完成编译后，生成可执行二进制文件`recipe_app`，然后使用如下命令执行它：

`$ . /recipe_app `

书中很多例子，都是通过标准输入读取整个文件的内容。遇到这样的例子时，我们使用标准UNIX管道和`cat`命令直接将文件内容传输给我们的应用，命令如下所示：

`$ cat file.txt | ./recipe_app`

上面的方法适用于Linux和Mac OS系统。在微软Windows Shell中，需要使用如下的命令: 

`> recipe app.exe < file.txt`

如果你不想在Shell命令行中运行，你可以在Microsoft Visual Studio IDE中运行，不过需要你修改一下配置, `"Configuration properties > Debugging "`,并且添加`"< file. txt"` ，使用 Visual Studio加载应用就能直接运行程序了。(Visual Studio IDE的话选定对应的解决方案，右键后选择“属性”，在“调试”页面输入相应的命令行参数)

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