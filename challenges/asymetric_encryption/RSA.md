# RSA
Task: Your task is to find the KEY and try to decrypt given RSA encrypted flag.

## Static analysis
Clicking on the "CHECK RSA FLAG" button results in a base64 encoded encrypted value.

![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/asymetric_encryption/img/encrypted_flag.png?raw=true)

```
echo "OplOuJowyt9ZtkIPKcmi6FVp50sZGd3+TbwXgQYxW0v/+sA87tG2ViJ9D5GOFHvl7nffGNLju5kzE33b1CKztu6/rHbIsu5lftp2qgtLQZdIdYy7F6MDhbvyLNk786QgXPLwND6ccFUv4brz8GAAefougdBYKr9o8IWBw8HxRfu884WLQnOxuhMnANXOU7yp1xaap2ojejSeNNbOWn1VKBMaZviXGVXXR6m4lFlHxMUXEP9h4bjrMraapiNLMM+2sUCuhD80beZK1NVakchy8FpqfJ/+hXaokSvnvQIHTr3H9JPD+flzwgYQn9FVvet4494+c2+rBTFOMdrYxHQwqg==" | base64 -d | hd
00000000  3a 99 4e b8 9a 30 ca df  59 b6 42 0f 29 c9 a2 e8  |:.N..0..Y.B.)...|
00000010  55 69 e7 4b 19 19 dd fe  4d bc 17 81 06 31 5b 4b  |Ui.K....M....1[K|
00000020  ff fa c0 3c ee d1 b6 56  22 7d 0f 91 8e 14 7b e5  |...<...V"}....{.|
00000030  ee 77 df 18 d2 e3 bb 99  33 13 7d db d4 22 b3 b6  |.w......3.}.."..|
00000040  ee bf ac 76 c8 b2 ee 65  7e da 76 aa 0b 4b 41 97  |...v...e~.v..KA.|
00000050  48 75 8c bb 17 a3 03 85  bb f2 2c d9 3b f3 a4 20  |Hu........,.;.. |
00000060  5c f2 f0 34 3e 9c 70 55  2f e1 ba f3 f0 60 00 79  |\..4>.pU/....`.y|
00000070  fa 2e 81 d0 58 2a bf 68  f0 85 81 c3 c1 f1 45 fb  |....X*.h......E.|
00000080  bc f3 85 8b 42 73 b1 ba  13 27 00 d5 ce 53 bc a9  |....Bs...'...S..|
00000090  d7 16 9a a7 6a 23 7a 34  9e 34 d6 ce 5a 7d 55 28  |....j#z4.4..Z}U(|
000000a0  13 1a 66 f8 97 19 55 d7  47 a9 b8 94 59 47 c4 c5  |..f...U.G...YG..|
000000b0  17 10 ff 61 e1 b8 eb 32  b6 9a a6 23 4b 30 cf b6  |...a...2...#K0..|
000000c0  b1 40 ae 84 3f 34 6d e6  4a d4 d5 5a 91 c8 72 f0  |.@..?4m.J..Z..r.|
000000d0  5a 6a 7c 9f fe 85 76 a8  91 2b e7 bd 02 07 4e bd  |Zj|...v..+....N.|
000000e0  c7 f4 93 c3 f9 f9 73 c2  06 10 9f d1 55 bd eb 78  |......s.....U..x|
000000f0  e3 de 3e 73 6f ab 05 31  4e 31 da d8 c4 74 30 aa  |..>so..1N1...t0.|
00000100
```

The relevant classes can be found with a simple search in jadx-gui for the string "RSA/"

![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/asymetric_encryption/img/rsa_relevant_classes.png?raw=true)

As it can be seen the class **com.hpandro.androidsecurity.ui.activity.task.encryption.RSAActivity** contains a **"decrypt"** method which takes the private key and the encrypted value as arguments and returns the decrypted value as a byte array.
![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/asymetric_encryption/img/rsa_enc_dec_methods.png?raw=true)

## Dynamic analysis
The simplest method to extract the private key and the decrypted flag is to use [objection](https://github.com/sensepost/objection) to hook on the **"decrypt"** method of the class **com.hpandro.androidsecurity.ui.activity.task.encryption.RSAActivity**

### Run [frida server](https://github.com/frida/frida/releases) on the emulator
```
$ adb push frida-server-15.0.19-android-x86 /data/local/tmp/frida-srv
frida-server-15.0.19-android-x86: 1 file pushed, 0 skipped. 62.5 MB/s (46351796 bytes in 0.708s)
$ adb shell
vbox86p:/ # cd /data/local/tmp
vbox86p:/data/local/tmp # chmod +x frida-srv && ./frida-srv
```

### Objection hook
```
$ objection -g "com.hpandro.androidsecurity" explore                                                                                                                  
com.hpandro.androidsecurity on (Android: 10) [usb] # android hooking watch class_method com.hpandro.androidsecurity.ui.activity.task.encryption.RSAActivity.decrypt --dump-args --dump-return
```

When the "CHECK RSA FLAG" button is clicked we'll see the following output in objection:
```
(agent) [076864] Arguments com.hpandro.androidsecurity.ui.activity.task.encryption.RSAActivity.decrypt([object Object], 58,-103,78,-72,-102,48,-54,-33,89,-74,66,15,41,-55,-94,-24,85,105,-25,75,25,25,-35,-2,77,-68,23,-127,6,49,91,75,-1,-6,-64,60,-18,-47,-74,86,34,125,15,-111,-114,20,123,-27,-18,119,-33,24,-46,-29,-69,-103,51,19,125,-37,-44,34,-77,-74,-18,-65,-84,118,-56,-78,-18,101,126,-38,118,-86,11,75,65,-105,72,117,-116,-69,23,-93,3,-123,-69,-14,44,-39,59,-13,-92,32,92,-14,-16,52,62,-100,112,85,47,-31,-70,-13,-16,96,0,121,-6,46,-127,-48,88,42,-65,104,-16,-123,-127,-61,-63,-15,69,-5,-68,-13,-123,-117,66,115,-79,-70,19,39,0,-43,-50,83,-68,-87,-41,22,-102,-89,106,35,122,52,-98,52,-42,-50,90,125,85,40,19,26,102,-8,-105,25,85,-41,71,-87,-72,-108,89,71,-60,-59,23,16,-1,97,-31,-72,-21,50,-74,-102,-90,35,75,48,-49,-74,-79,64,-82,-124,63,52,109,-26,74,-44,-43,90,-111,-56,114,-16,90,106,124,-97,-2,-123,118,-88,-111,43,-25,-67,2,7,78,-67,-57,-12,-109,-61,-7,-7,115,-62,6,16,-97,-47,85,-67,-21,120,-29,-34,62,115,111,-85,5,49,78,49,-38,-40,-60,116,48,-86)
(agent) [076864] Return Value: 104,112,97,110,100,114,111,123,82,83,65,46,111,115,79,48,76,53,67,71,54,90,101,120,49,104,111,57,113,121,54,69,71,102,102,56,85,106,86,100,69,87,56,72,125
```

(Note: The Java byte type is a signed integer; the value ranges between -128 and 127.)

The following **Java snippet** can be used to convert the byte array into a string:
```
public class ByteArraytoString
{
        public static void main(String[] args)
                {

                        byte[] byteArray = {104,112,97,110,100,114,111,123,82,83,65,46,111,115,79,48,76,53,67,71,54,90,101,120,49,104,111,57,113,121,54,69,71,102,102,56,85,106,86,100,69,87,56,72,125};
                        String str = new String(byteArray);
                        System.out.println(str);
                }
}
```
```
$ javac ByteArraytoString.java && java ByteArraytoString
hpandro{REDACTED}
```

The following **Python snippet** can be used to convert the byte array into string:
```
$ python3
>>> items = [104,112,97,110,100,114,111,123,82,83,65,46,111,115,79,48,76,53,67,71,54,90,101,120,49,104,111,57,113,121,54,69,71,102,102,56,85,106,86,100,69,87,56,72,125]
>>> data = bytes(b % 256 for b in items)                                                     
>>> print(data)
b'hpandro{REDACTED}'
```

### Frida hook


# Reference
* https://stackoverflow.com/questions/40951213/how-to-convert-java-byte-to-python-string