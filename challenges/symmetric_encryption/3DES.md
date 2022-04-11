# 3DES
Task: Your task is to find KEY and try to decrypt given 3DES encrypted flag.

## Static analysis
Clicking on the "CHECK 3DES FLAG" button results in a base64 encoded encrypted value.

![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/symmetric_encryption/img/3des_enc_flag?raw=true)

```
echo "vhN2YEWA/njdWjOto6lT7FYaejL96ED/Ra5d7PcHtr303nSTl07llWhNFI1tfMWJ4OuTet6EWsc=" | base64 -d | hd
00000000  be 13 76 60 45 80 fe 78  dd 5a 33 ad a3 a9 53 ec  |..v`E..x.Z3...S.|
00000010  56 1a 7a 32 fd e8 40 ff  45 ae 5d ec f7 07 b6 bd  |V.z2..@.E.].....|
00000020  f4 de 74 93 97 4e e5 95  68 4d 14 8d 6d 7c c5 89  |..t..N..hM..m|..|
00000030  e0 eb 93 7a de 84 5a c7                           |...z..Z.|
00000038
```