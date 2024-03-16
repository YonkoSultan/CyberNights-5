After extracting the zip file, we can obviously see a `pcap` file, which stores captured network packets. So we will use wireshark.

During previewing the packets we notice that there is a few interesting `HTTP` packets.

By writting `http` in the display filter,
<br>
we get 61 http packets that contains requests to `images, and flagyard & kali-linux sites`.

![Screenshot (36)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/0c893f5e-7652-41ae-93c6-24425ab45e04)


After following the HTTP stream, we end up with 4 interesting requests, 3 of them contains `weird text`.


![Screenshot (38)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/9932e432-025b-47e7-8ec6-7ed7f7991cd7)
![Screenshot (39)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/8d371fad-0d4a-4cfb-a9c4-042aa8c39cf2)
![Screenshot (40)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/9b224384-7a0e-4028-a97b-31f794c58491)

and another request that contains `?whythathappen`.

Going to https://www.dcode.fr/cipher-identifier to identify which encoding or encryption is it

![Screenshot (42)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/206bcf0b-0226-41e3-a75d-26035a976acb)


Ok, so it's a base64, after decoding the first request's encoded text in CyberChef, we get :


![Screenshot (44)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/cef8c6d7-e1b2-4396-b391-da917af4f9f4)


So it's a `zip` file header, assuming that other requests are the data of the zip file.
Decoding them & concatenating them in order, then downloading the result zip file and then try to extract it we get this error:

![Screenshot (45)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/c4b3673b-7f5d-4db5-be37-26fdd3599db3)


which indicates the AES (Adavance Encryption Standard) encryption. Unfortunately, This encryption standard is currently not supported by unzip binary. However, 7zip package can be used to extract such files.
but when we try to extract it, we get a password prompt : 

![Screenshot (46)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/5bde24a7-ec5f-4259-9ddc-f917f2c1a1ba)

Trying `whythathappen` won't work, if we return to the packets we remember that there were images, so maybe the password is there!
exporting the images by going to `File >> Export Objects >> HTTP` and then pressing `Save All`, we get them all in our saved directory.

The challenge is `Forensics` so you know what to do :)

Applying `steghide`, `strings`, `foremost`, `binwalk`, `exiftool` tools didn't result in anything,
but when we try [stegolve](https://github.com/manisashank/stegsolve/blob/master/process%20to%20install%20stegsolve) on `image.png` specially at Red plane 1; there is a QR code! :

![Screenshot (50)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/d28c8c8f-fbfa-4786-86e9-a3cf602486e1)

Using your phone camera or any QR code scanner, you will get `Passw0rd IS : SUP3RM4N_P4SS`.

After extracting the zip file, we get `private.wav` when opening it with audio player, there's nothing except a tone.

Opening it with `Audacity` we get :

![Screenshot (51)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/da7c9914-7dcb-4de9-a174-cd9d11a363e5)

If we focus on the waves, we see two types of them :

- Big wave
- Small wave

And there are periods between them, zooming in a little bit :

![Screenshot (53)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/62479ff7-dbd0-408b-8c8c-c058d97cc875)

It's obviously `Morse code`!

Big waves for dashes `-`
Small waves for dots `.`
Periods are spaces

So if we try to decode the first few waves (I used this site to translate the dashes and dots [Morse code decoder](https://morsecode.world/international/translator.html)) :

![Screenshot (53)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/b23cfeaf-9632-4fbb-bbce-df39aceba077)

Ok, now everything is obvious, you either decode it manually or code a program that automatically decodes it, or upload it to a website that will translate the waves into dashes and dots and then decodes it for you, like : [Data border](https://databorder.com/transfer/morse-sound-receiver/)

![Screenshot (54)](https://github.com/SultanCYB/CyberNights-5/assets/107263975/a530ee8b-0acb-4052-9a38-081da43bdd80)

Easy peasy lemon squeezy!
The flag format is `FLAGY{}` so just wrap it with the curly braces and you are done.
