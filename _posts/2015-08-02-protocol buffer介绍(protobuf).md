---
layout: post
tile:  "protocol buffer介绍(protobuf)"
date:  2015-08-02 08:30:57
categories: hadoop 大数据 
excerpt: protocol buffer介绍(protobuf)
---

* content
{:toc}



一、理论概述

0、参考资料

入门资料：https://developers.google.com/protocol-buffers/docs/javatutorial

更详细的资料：

For more detailed reference information, see the Protocol Buffer Language Guide, the Java API Reference, the Java Generated Code Guide, and the Encoding Reference.


1、protobuf是什么？

看看官方的解释

   Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.
   
也就是说：protobuf是一个google的开源项目，它是一个语言独立、平台独立，可扩展的数据序列化机制，类似于XML，但它更小、更快和更简单。

2、protobuf能做什么？

很明显，它可用于对象序列化与反序列化，主要用于数据存储与数据传输格式的定义。
目前被大量用于hadoop的RPC通信协议中，所有的RPC函数参数均是使用protobuf定义的。

3、与XML相比的优缺点

优点：更小，更快（XML的反序列化效率极低），而且可以利用工具自动生成代码。

缺点：由于用二进制保存数据，导致可读性差    

二、API

1、使用protobuf的基本步骤如下：

（1）定义消息的格式（一般使用.proto后缀）

（2）使用protobuf提供的compiler，根据.proto文件生成类

（3）使用API进行消息的读写

2、定义消息的格式（一般使用.proto后缀）

		 
	package tutorial;
	
	option java_package = "org.ljh.protobufdemo";
	option java_outer_classname = "AddressBookProtos";
	
	message Person {
	  required string name = 1;
	  required int32 id = 2;
	  optional string email = 3;
	
	  enum PhoneType {
	    MOBILE = 0;
	    HOME = 1;
	    WORK = 2;
	  }
	
	  message PhoneNumber {
	    required string number = 1;
	    optional PhoneType type = 2 [default = HOME];
	  }
	
	  repeated PhoneNumber phone = 4;
	}
	
	message AddressBook {
	  repeated Person person = 1;
	}
 
proto文件说明，完整说明请见https://developers.google.com/protocol-buffers/docs/proto
（1）package用于指明命名空间，以防与其它项目冲突。
（2）如果指定java_package，则它作为下一步要生成的java类的package，否则package中定义的值将作为java类的package。
（3）java_outer_classname指定了生成类的类名，如果没指定，则使用.proto文件的文件名作为类名。
（4）message表示消息定义，消息之间可以互相嵌套或者调用。
（5）每个字段后的等号定义的是该字段的tag，由于1～15少用了一个字节，因此，这些标签最好留给用得很多的字段，尤其是使用repeated定义的字段。
（6）每个字段必须使用以下3个修饰符之一：required, optional, repeated。

3、从标准输入中读入信息，构建person，然后序列化到一个文件中
		
	package org.ljh.protobufdemo;
	import java.io.BufferedReader;
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.FileOutputStream;
	import java.io.InputStreamReader;
	import java.io.IOException;
	import java.io.PrintStream;
	
	import org.ljh.protobufdemo.AddressBookProtos.AddressBook;
	import org.ljh.protobufdemo.AddressBookProtos.Person;
	
	public class AddPerson {
	  // This function fills in a Person message based on user input.
	  static Person PromptForAddress(BufferedReader stdin,
	                                 PrintStream stdout) throws IOException {
	    Person.Builder person = Person.newBuilder();
	
	    stdout.print("Enter person ID: ");
	    person.setId(Integer.valueOf(stdin.readLine()));
	
	    stdout.print("Enter name: ");
	    person.setName(stdin.readLine());
	
	    stdout.print("Enter email address (blank for none): ");
	    String email = stdin.readLine();
	    if (email.length() > 0) {
	      person.setEmail(email);
	    }
	
	    while (true) {
	      stdout.print("Enter a phone number (or leave blank to finish): ");
	      String number = stdin.readLine();
	      if (number.length() == 0) {
	        break;
	      }
	
	      Person.PhoneNumber.Builder phoneNumber =
	        Person.PhoneNumber.newBuilder().setNumber(number);
	
	      stdout.print("Is this a mobile, home, or work phone? ");
	      String type = stdin.readLine();
	      if (type.equals("mobile")) {
	        phoneNumber.setType(Person.PhoneType.MOBILE);
	      } else if (type.equals("home")) {
	        phoneNumber.setType(Person.PhoneType.HOME);
	      } else if (type.equals("work")) {
	        phoneNumber.setType(Person.PhoneType.WORK);
	      } else {
	        stdout.println("Unknown phone type.  Using default.");
	      }
	
	      person.addPhone(phoneNumber);
	    }
	
	    return person.build();
	  }
	
	  // Main function:  Reads the entire address book from a file,
	  //   adds one person based on user input, then writes it back out to the same
	  //   file.
	  public static void main(String[] args) throws Exception {
	    if (args.length != 1) {
	      System.err.println("Usage:  AddPerson ADDRESS_BOOK_FILE");
	      System.exit(-1);
	    }
	
	    AddressBook.Builder addressBook = AddressBook.newBuilder();
	
	    // Read the existing address book.
	    try {
	      addressBook.mergeFrom(new FileInputStream(args[0]));
	    } catch (FileNotFoundException e) {
	      System.out.println(args[0] + ": File not found.  Creating a new file.");
	    }
	
	    // Add an address.
	    addressBook.addPerson(
	      PromptForAddress(new BufferedReader(new InputStreamReader(System.in)),
	                       System.out));
	
	    // Write the new address book back to disk.
	    FileOutputStream output = new FileOutputStream(args[0]);
	    addressBook.build().writeTo(output);
	    output.close();
	  }
	}
	

4、从文件中读取内容，然后反序列化到一个实例
		
	package org.ljh.protobufdemo;
	
	
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.io.PrintStream;
	
	import org.ljh.protobufdemo.AddressBookProtos.AddressBook;
	import org.ljh.protobufdemo.AddressBookProtos.Person;
	
	public class ListPeople {
	  // Iterates though all people in the AddressBook and prints info about them.
	  static void Print(AddressBook addressBook) {
	    for (Person person: addressBook.getPersonList()) {
	      System.out.println("Person ID: " + person.getId());
	      System.out.println("  Name: " + person.getName());
	      if (person.hasEmail()) {
	        System.out.println("  E-mail address: " + person.getEmail());
	      }
	
	      for (Person.PhoneNumber phoneNumber : person.getPhoneList()) {
	        switch (phoneNumber.getType()) {
	          case MOBILE:
	            System.out.print("  Mobile phone #: ");
	            break;
	          case HOME:
	            System.out.print("  Hls"
	                    + "ome phone #: ");
	            break;
	          case WORK:
	            System.out.print("  Work phone #: ");
	            break;
	        }
	        System.out.println(phoneNumber.getNumber());
	      }
	    }
	  }
	
	  // Main function:  Reads the entire address book from a file and prints all
	  //   the information inside.
	  public static void main(String[] args) throws Exception {
	    if (args.length != 1) {
	      System.err.println("Usage:  ListPeople ADDRESS_BOOK_FILE");
	      System.exit(-1);
	    }
	
	    // Read the existing address book.
	    AddressBook addressBook =
	      AddressBook.parseFrom(new FileInputStream(args[0]));
	
	    Print(addressBook);
	  }
	}
