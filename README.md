# Coolstar Patched

### Legal Note:
This is a simple tutorial intended to share information on how reverse engineering a Windows driver can be done. I have legally purchased all the Coolstar drivers, as they have invested significant time and effort. You should respect that as well.

<hr/>
We are trying to break Intel TCSS licensing.

### Step 1: How the installer works?
The setup file content can be extracted using 7zip as below
<img src="img1.png">

It use dpinst to install the drivers and each folder contains *.cat, *.inf and *.sys, We are only intereseted in .sys file which contains the actual driver code.

But there are 2 driver intelpmc and inteltcss, Which is the main one? the pmc driver doesnt check the license it just obersce the status on tcss driver so that is the only driver reads out license file.

`I'm going to use binary ninja which is the most underrated binary reverse engieering tool`

After opening the file we first would check the string to see if we can find any refrence to the license file name `inteltcss_signedlicense.bin` but no results!

Now I have to check for a function that reads a file and after checking the symbols we hit the jackpot, `ZwReadFile` is exactly what we are looking for

<img src="img2.png">

And after checking the code refrence we find a function that only check license file status and it's content so nothing intersting there

<img src="img3.png">

But when checking the refrence of parent function we found on of the longest code of our file that checks the license content by first double hash the data and then use some sort og ECDSA verification then checks if the data match HWID and returns 0 on success

<img src="img4.png">

Now all we had to do is never branch the first `if` statement

<img src="img5.png">

Then change return value to 0x0, Now our driver file is ready!

<img src="img6.png">

How to test if it works? We should put windwos in `testsigning` mode since MS doesnt allow for signed drivers without a trusted CA, After that we should self sign out driver file:

```sh
openssl req -new -x509 -days 3650 -keyout test.key -out test.crt
openssl pkcs12 -export -out test.pfx -inkey test.key -in test.crt
osslsigncode sign -pkcs12 test.pfx -pass password -n "password" -in inteltcss.sys -out inteltcss_patched.sys
```

Now you need to replace the patched driver with the real file in `C:/Windows/System32/drivers`.

Restart and see the magic :)