---
layout: post
comments: true
title: Data transfer between HM-10 and Android
tags: [Android, Arduino, Bluetooth]
---

HM-10 bluetooth module is the most common BT module for Andruino. Chances are that if you're working on sending or receiving data via bluetooth with Arduino you'll be using the HM-10 module. 

Now, there are loads of articles and tutorials directing how to make an app which will connect to the module and send/ receive data hence I won't be rewriting that. What this post is about how I overcame a problem with respect to data transfer between HM-10 and Android app with a simple fix.

**Problem** - Data being accepted getting jumbled up to the point where the data made no sense. Example, the module emits data like this - 
`0,501,486,995;` But the android device receives the data like this - `,496,482,995;0,503,488,995;0,501,486,995;0,496,480,996;0,502,486,995;0,` or `996;,474,`.

The code to accept data is usually similar to this - 

    private void getDataFromDevice() {

    	byte[] buffer = new byte[256];  // buffer store for the stream
    	int bytes; // bytes returned from read()
    	int length;
    		try {

        		InputStream tmpIn = null;
        		OutputStream tmpOut = null;

        		// Get the BluetoothSocket input and output streams
        		tmpIn = bTSocket.getInputStream();
        		tmpOut = bTSocket.getOutputStream();

        		DataInputStream mmInStream = new DataInputStream(tmpIn);
        		DataOutputStream mmOutStream = new DataOutputStream(tmpOut);

        		// Read from the InputStream
        		while(true) {

            		bytes = mmInStream.read(buffer);
            		length = mmInStream.read(buffer);

            		String readMessage = new String(buffer, 0, bytes);
            		// Send the obtained bytes to the UI Activity

            		Log.i("DATA---", readMessage);
        		}

    		} catch (IOException ex) {
        		ex.printStackTrace();
        		cancel();
    	}
	}

The variable `buffer` instructs how many bytes to read from the stream at a time. This is what the source of problem was in my case; reducing it to a single byte did the trick for me. Hence, `byte[] buffer = new byte[1];` fixed the sequence of the input stream.

