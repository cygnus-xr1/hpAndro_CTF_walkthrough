# 3DES
Task: Your task is to find KEY and try to decrypt given 3DES encrypted flag.

## Static analysis
Clicking on the "CHECK 3DES FLAG" button results in a base64 encoded encrypted value.

![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/symmetric_encryption/img/3des_enc_flag.png?raw=true)

```
echo "vhN2YEWA/njdWjOto6lT7FYaejL96ED/Ra5d7PcHtr303nSTl07llWhNFI1tfMWJ4OuTet6EWsc=" | base64 -d | hd
00000000  be 13 76 60 45 80 fe 78  dd 5a 33 ad a3 a9 53 ec  |..v`E..x.Z3...S.|
00000010  56 1a 7a 32 fd e8 40 ff  45 ae 5d ec f7 07 b6 bd  |V.z2..@.E.].....|
00000020  f4 de 74 93 97 4e e5 95  68 4d 14 8d 6d 7c c5 89  |..t..N..hM..m|..|
00000030  e0 eb 93 7a de 84 5a c7                           |...z..Z.|
00000038
```

A quick grep for the button's text shows us the relevant activity's name we should check further in Jadx-GUI
```
$ grep -ri "CHECK 3DES FLAG"
com.hpandro.androidsecurity_1.2/res/layout/activity_three_des.xml:        <Button android:textAppearance="@style/TextAppearance.AppCompat.Button" android:textColor="@android:color/white" android:id="@id/btnCheckEncryptFlag" android:background="@drawable/btn_drawable" android:paddingLeft="20.0dip" android:paddingRight="20.0dip" android:layout_width="wrap_content" android:layout_height="wrap_content" android:layout_marginTop="16.0dip" android:minWidth="150.0dip" android:text="Check 3DES Flag" />
```

These are the relevant classes we should analyze:
![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/symmetric_encryption/img/3des_jadx_classes.png?raw=true)

The **onGetLogs** method of the class **com.hpandro.androidsecurity.ui.activity.task.encryption.ThreeDESActivity** shows that a hardcoded password is used as the encryption key (3des_enc_pass).
![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/symmetric_encryption/img/3des_onGetLogs.png?raw=true)

A quick grep reveals the hardcoded password used for the DES3 encryption:
```
$ grep -ri "3des_enc_pass"
res/values-ar-rXB/strings.xml:    <string name="_3des_enc_pass">‏‮3hpAndro‬‏</string>
res/values/public.xml:    <public type="string" name="_3des_enc_pass" id="0x7f100011" />
res/values/strings.xml:    <string name="_3des_enc_pass">3hpAndro</string>
```

The **decrypt** method takes 2 parameters, the encrypted data (str) and the hardcoded password (str2).
![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/symmetric_encryption/img/3des_decrypt_method.png?raw=true)

As it can be seen above, the key used for encryption/decryption is the first 24 bytes of the hardcoded password's SHA1 hash (line 83,84,88).

The following Java snippet is the replication of the "decrypt" method found in class **com.hpandro.androidsecurity.ui.activity.task.encryption.ThreeDESActivity**:
```
import java.util.Arrays;
import java.util.Base64;
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.MessageDigest;


public class threeDES_decrypt {

    private String decrypt(String data, String secretKey) {
        try {
            // Base64 decode && Get the bytes of the encrypted data
            byte[] encBytes = Base64.getDecoder().decode(data.getBytes("utf-8"));

            // Get the SHA1 digest of the secretKey
            MessageDigest instance = MessageDigest.getInstance("SHA1");
            byte[] bytePass = instance.digest(secretKey.getBytes("utf-8"));
            
            // First 24 bytes of the SHA1 hash
            byte[] byteKey = Arrays.copyOf(bytePass, 24);
            
            // Decryption
            SecretKey key = new SecretKeySpec(byteKey, "DESede");
            Cipher cipher = Cipher.getInstance("DESede");
            cipher.init(Cipher.DECRYPT_MODE, key);
            byte[] byteText = cipher.doFinal(encBytes);
            String clearText = new String(byteText, "utf-8");
            
            return clearText;
        }
        catch(Exception ex) {
            return ex.getMessage();
        }
    }
 
    public static void main(String[] args) {
       
        String data = "";
        String secretKey = "";
        String clearText = new threeDES_decrypt().decrypt(data, secretKey);

        System.out.println(clearText);
    }
}
```

```
$ javac threeDES_decrypt.java && java threeDES_decrypt
hpandro{REDACTED}
```