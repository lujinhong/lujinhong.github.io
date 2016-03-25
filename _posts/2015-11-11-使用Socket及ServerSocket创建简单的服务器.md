---
layout: post
tile:  "使用Socket及ServerSocket创建简单的服务器"
date:  2015-11-11 18:34:16
categories: java 
excerpt: 使用Socket及ServerSocket创建简单的服务器
---

* content
{:toc}



  参考自core java

	package com.lujinhong.corejava;
	
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.io.PrintWriter;
	import java.net.ServerSocket;
	import java.net.Socket;
	import java.util.Scanner;
	
	public class EchoServer {
	
		public static void main(String[] args) {
			
			try {
				ServerSocket serverSocket = new ServerSocket(8019);
				Socket socket = serverSocket.accept();
				
				InputStream is = socket.getInputStream();
				OutputStream os = socket.getOutputStream();
				
				PrintWriter pw = new PrintWriter(os);
				Scanner sc = new Scanner(is);
				
				Boolean flag = false;
				String line = null;
				String exitString = "bye";
				
				while(!flag && sc.hasNextLine()){
					pw.println("Hello, type " + exitString + " to exit!");
					line = sc.nextLine();
					if(line.trim().equals(exitString)){
						flag = true;
					}else{
						pw.println("Hello, "+line);
					}
				}
				pw.close();
				sc.close();
				serverSocket.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

 

使用线程实现多个客户端同时访问：

 

	package com.lujinhong.corejava;
	
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.io.PrintWriter;
	import java.net.ServerSocket;
	import java.net.Socket;
	import java.util.Scanner;
	
	public class MultiEchoServer {
	
		public static void main(String[] args) {
			try {
				ServerSocket serverSocket = new ServerSocket(8189);
				while (true) {
					Socket socket = serverSocket.accept();
	
					Runnable r = new ThreadedEchoHandler(socket);
					Thread thread = new Thread(r);
					thread.start();
				}
	
			} catch (IOException e) {
				e.printStackTrace();
			}
	
		}
	
	}
	
	class ThreadedEchoHandler implements Runnable {
	
		private Socket s = null;
	
		public ThreadedEchoHandler(Socket socket) {
			s = socket;
		}
	
		@Override
		public void run() {
			try {
				InputStream is = s.getInputStream();
				OutputStream os = s.getOutputStream();
	
				PrintWriter pw = new PrintWriter(os);
				Scanner sc = new Scanner(is);
	
				Boolean flag = false;
				String line = null;
				String exitString = "bye";
	
				while (!flag && sc.hasNextLine()) {
					pw.println("Hello, type " + exitString + " to exit!");
					line = sc.nextLine();
					if (line.trim().equals(exitString)) {
						flag = true;
					} else {
						pw.println("Hello, " + line);
					}
				}
				sc.close();
				pw.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}


In this program, we spawn a separate thread for each connection. This approach is not satisfactory for highperformance servers. You can achieve greater server throughput by using features of the java.nio package. See www.ibm.com/developerworks/java/library/j-javaio for more information.
