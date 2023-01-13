<p align="center">
  THM : Confidential<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/Qnvqtg2.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [Get the QR Code](#get-the-qr-code)
- [Conclusion](#conclusion)

## Introduction

Let's read the instructions of this room :  
```
We got our hands on a confidential case file from some self-declared "black hat hackers"... it looks like they have a secret invite code available within a QR code, 
but it's covered by some image in this PDF! If we want to thwart whatever it is they are planning, we need your help to uncover what that QR code says!

Access this challenge by deploying the machine attached to this task by pressing the green "Start Machine" button. This machine shows in Split View in your browser, 
if it doesn't automatically display you may need to click "Show Split View" in the top right.

The file you need is located in /home/ubuntu/confidential on the VM.
```

## Get the QR Code

Let's take a look at the given pdf file :  
![](https://i.imgur.com/3LktqiL.jpg)  

So we need to uncover the QR Code. It seems that one image (the red sign) was placed over the other (the QR Code). If it's two separate images, it's maybe possible to 
extract them from the pdf file. After some research, I found that you can extract images from a pdf file using `pdfimages` from [Poppler-utils](https://doc.ubuntu-fr.org/poppler-utils#poppler-utils).
Let's use this tool to extract images from the pdf file :  
```
ubuntu@thm-confidential:~/confidential$ pdfimages Repdf.pdf image
ubuntu@thm-confidential:~/confidential$ ls
Repdf.pdf  image-000.ppm  image-001.ppm  image-002.ppm
```

We successfuly extracted three images from the pdf file :  
![](https://i.imgur.com/EvA5Uf4.jpg)

Now we can read the QR Code. But be careful, you should never scan an unknown QR Code, you can use online tools for this. I used [Web QR](https://webqr.com/index.html) :  
![](https://i.imgur.com/uDhgqeW.jpg)

And we have the flag !

## Conclusion

What I've learned in this room is that it's possible to extract images from a pdf files and see hidden content if the images was originally separated. Thanks for this room ! 
And thanks for reading my write ups !
