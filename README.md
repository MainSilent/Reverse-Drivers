# Coolstar Patched

### Legal  Note:
This is just a simple tutorial to share information on how reverse engineering a windows driver can be done, I have legally bought all the coolstar drivers as they have spent so much time and effort and you should also respect that.

<hr/>
We are trying to break intel tcss licensing.

### Step 1: How the installer works?
The setup file content can be extracted using 7zip as below
<img src="img1.png">

It use dpinst to install the drivers and each folder contains *.cat, *.inf and *.sys, We are only intereseted in .sys file which contains the actual driver code.

`I'm going to use binary ninja which is the most underrated binary reverse engieering tool`

