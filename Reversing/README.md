# REV

## BabyREV
Tại hàm main, có thể dễ dàng thấy được chương trình như thế nào.
```c
v12 = __readfsqword(0x28u);
  fgets(s, 64, stdin);
  s[strcspn(s, "\n")] = 0;
  v7 = strlen(s);
  n = 7LL;
  if ( strncmp("corctf{", s, 7uLL) )
    goto LABEL_12;
  if ( s[v7 - 1] != '}' )
    goto LABEL_12;
  if ( v7 != 28 )
    goto LABEL_12;
  memcpy(dest, &s[n], 28 - n - 1);
  dest[28 - n - 1] = 0;
  for ( i = 0; i < strlen(dest); ++i )
  {
    v6 = 4 * i;
    for ( j = is_prime(4 * i); j != 1; j = is_prime(v6) )
      ++v6;
    s1[i] = rot_n(dest[i], v6);
  }
  s1[strlen(s1) + 1] = 0;
  memfrob(check, 0x14uLL);
  if ( !strcmp(s1, check) )
  {
    puts("correct!");
    return 0;
  }
  else
  {
LABEL_12:
    puts("rev is hard i guess...");
    return 1;
  }
```
Ban đầu sẽ so sánh 7 kí tự đầu với `corctf{` sau đó sẽ so sánh kí tự cuối cùng với `}`, ngoài ra còn cho biết thêm chuỗi input có độ dài là 28
Tại vòng `for`, sẽ tính toán v6 để làm `key`, sau đó sẽ encrypt mỗi kí tự input với rot bằng key vừa tính được.
Code tìm key:
```c
int is_prime(int a1)
{
  int i; // [rsp+1Ch] [rbp-4h]

  if ( a1 <= 1 )
    return 0LL;
  for ( i = 2; i <= (int)sqrt((double)a1); ++i )
  {
    if ( !(a1 % i) )
      return 0;
  }
  return 1;
}
int main()
{
  	int v6;
  	for (int i = 0; i < 20; ++i )
  	{
	    v6 = 4 * i;
	    for (int j = is_prime(4 * i); j != 1; j = is_prime(v6) )
	      ++v6;
	    printf("%d ",v6);
    }
}
```
Tìm được key : `2 5 11 13 17 23 29 29 37 37 41 47 53 53 59 61 67 71 73 79`
Từ key này, dễ dàng code để tìm được flag.
```c
#include <iostream>
#include <string>
char ASCII_LOWER[] = "abcdefghijklmnopqrstuvwxyz";
char ASCII_UPPER[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
void Decrypt(char c, int key)
{
	int temp=0;
	temp = (int)c;
	if (temp == 95)
	{
		printf("_");
	}
	else if (temp >= 65 && temp <= 90)
	{
		temp %= 65;
		temp = (temp - key) % 26;
		if (temp < 0) temp += 26; 
		printf("%c",ASCII_UPPER[temp]);
	}
	else if (temp >= 97 && temp <= 122)
	{
		temp %= 97;
		temp = (temp - key) % 26;
		if (temp < 0) temp += 26; 
		printf("%c",ASCII_LOWER[temp]);
	}
	else
	{
		printf("%c",c);
	}
}
int main()
{
	char cipher[] = "ujp?_oHy_lxiu_zx_uve";
	int key[20] = {2, 5, 11, 13, 17, 23, 3, 3, 11, 11, 15, 21, 1, 1, 7, 9, 15, 19, 21, 1};
	int i;
	for ( i = 0; i < 20; ++i )
  	{
    	Decrypt(cipher[i], key[i]);
  	}
}
```
=> see?_rEv_aint_so_bad
### => Flag: `corctf{see?_rEv_aint_so_bad}`


## AliceInCeptionland
Một chall game khá thú vị.
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img1.png)

Vì file viết bằng `.NET` nên sử dụng dnspy để đọc. Game sẽ cho ta đi làm quest.
Tại quest đầu tiên (quest con mèo).
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img2.png)

Và đây là đoạn code của quest đó: 
```c#
private void button1_Click(object sender, EventArgs e)
{
    string text = WhiteRabbit.Transform("41!ce1337");
    char[] array = WhiteRabbit.Transform(this.textBox1.Text).Reverse<char>().ToArray<char>();
    for (int i = 0; i < array.Length; i++)
    {
        char[] array2 = array;
        int num = i;
        array2[num] ^= text[i % text.Length];
    }
    if (string.Join<char>("", array.Reverse<char>()).Equals("oI!&}IusoKs ?Ytr"))
    {
        this.Secret = this.textBox1.Text;
        this.label1.Visible = false;
        this.textBox1.Visible = false;
        this.button1.Visible = false;
        this.timer1.Start();
    }
}
```
Hàm này sẽ hoạt động khi nhấn nút trong quest, nó sẽ lấy chuỗi input và đảo ngược lại sau đó XOR với chuỗi `text` sau khi XOR xong sẽ đảo ngược và so sánh với `"oI!&}IusoKs ?Ytr"`.
Code lấy đáp án:
```c
#include <iostream>
#include <string.h>
int main()
{
	char s[] = "41!ce1337";
	char cipher[] = "rtY? sKosuI}&!Io";
	for (int i =0 ; i<strlen(cipher); i++)
	{
		cipher[i] ^= s[i%strlen(s)];
	}
	printf("%s", cipher);
}
```
=> FEx\EBx\DAx\EDx\
Đảo ngược lại sẽ ta sẽ có được câu trả lời của Quest còn mèo: `\xDE\xAD\xBE\xEF`
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img3.png)

Tiếp tục quest tiếp theo (quest con sâu):

![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img4.png)

Code của quest con sâu:
```c#
private byte rol(byte v, byte s)
{
    byte b = s % 8;
    return (byte)((int)v << (int)b | v >> (int)(8 - b));
}

private void button1_Click(object sender, EventArgs e)
{
    string text = WhiteRabbit.Transform("c4t3rp1114rz_s3cr3t1y_ru13_7h3_w0r1d");
    char[] array = WhiteRabbit.Transform(this.textBox1.Text).Reverse<char>().ToArray<char>();
    for (int i = 0; i < array.Length; i++)
    {
        byte b = Convert.ToByte(array[i]);
        b = this.rol(b, 114);
        b += 222;
        b ^= Convert.ToByte(text[i % text.Length]);
        b -= 127;
        b = this.rol(b, 6);
        array[i] = Convert.ToChar(b);
    }
    if (string.Join<char>("", array.Reverse<char>()).Equals("\0R\u009c\u007f\u0016ndC\u0005î\u0093MíÃ×\u007f\u0093\u0090\u007fS}­\u0093)ÿÃ\f0\u0093g/\u0003\u0093+Ã¶\0Rt\u007f\u0016\u0087dC\aî\u0093píÃ8\u007f\u0093\u0093\u007fSz­\u0093ÇÿÃÓ0\u0093\u0086/\u0003q"))
    {
        this.Secret = this.textBox1.Text;
        this.label1.Visible = false;
        this.textBox1.Visible = false;
        this.button1.Visible = false;
        this.timer1.Start();
    }
}
```
Hàm `button1_Click` hoạt động như ở hàm trên, sẽ lấy chuỗi input và đảo ngược sau đó trải qua một quá trình biến đổi cuối cùng đảo ngược lại và so sánh với chuỗi.
Code lấy đáp án:
```c#
private static byte ror(byte v, byte s)
{
    byte b = (byte)(s % 8);
    return (byte)(v >> (int)b | (int)v << (int)(8 - b));
}
private static void q2()
{
    String result = "";
    string text = "c4t3rp1114rz_s3cr3t1y_ru13_7h3_w0r1d";
    string array = "\0R\u009c\u007f\u0016ndC\u0005î\u0093MíÃ×\u007f\u0093\u0090\u007fS}­\u0093)ÿÃ\f0\u0093g/\u0003\u0093+Ã¶\0Rt\u007f\u0016\u0087dC\aî\u0093píÃ8\u007f\u0093\u0093\u007fSz­\u0093ÇÿÃÓ0\u0093\u0086/\u0003q";
    char[] arr = array.ToCharArray();
    string tmp = string.Join<char>("", arr.Reverse<char>());
    arr = tmp.ToCharArray();
    for (int i = 0; i < arr.Length; i++)
    {
        byte b = Convert.ToByte(arr[i]);
        b = ror(b, 6);
        b += 127;
        b ^= Convert.ToByte(text[i % text.Length]);
        b -= 222;
        b = ror(b, 114);
        arr[i] = Convert.ToChar(b);
        result += arr[i].ToString();
    }
    Console.Write(string.Join<char>("", result.ToCharArray().Reverse<char>()));
}
static void Main(string[] args)
{
    q2();
}
```
=> Đáp án: \x4\xL\x1\xC\x3\x1\xS\xN\x0\xT\x4\xS\xL\x3\x3\xP\xS\x4\xV\x3\xH\x3\xR
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img5.png)

Sau khi giết được con sâu, sẽ đi theo Alice. Lưu ý: sẽ thấy được flag rất dễ dàng, nhưng đó là flag pha ke.
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img5-1.png)

Khi chúng ta chọn theo Alice, chương trình sẽ load một số đoạn code từ resources của game (Class program)
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img6.png)
và sử dụng vào quest cuối cùng (dòng 28, sẽ gọi một hàm `Encode` của class `Dream` nằm trong `AliceInCeptiondream`).
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img7.png)

Tiến hành debug và đặt breakpoint tại nơi nó đã load được từ resouces (cụ thể là hàm `button1_click`) và có thể thấy được `AliceInCeptiondream`
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img8.png)

Xem chi tiết `AliceInCeptiondream`:
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img9.png)

Sau một hồi phân tích hàm encode, tôi cảm thấy mất thời gian nên sẽ tiến hành sử dụng hàm encode để brute force lấy đáp án.
Code brute force:
```c#
static void Main(string[] args)
{
	String s = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!#@$%&*()^ ";
	String cipher = "3c3cf1df89fe832aefcc22fc82017cd57bef01df54235e21414122d78a9d88cfef3cf10c829ee32ae4ef01dfa1951cd51b7b22fc82433ef7ef418cdf8a9d802101ef64f9a495268fef18d52882324f217b1bd64b82017cd57bef01df255288f7593922712c958029e7efccdf081f8808a6efd5287595f821482822f6cb95f821cceff4695495268fefe72ad7821a67ae0060ad";

	char[] ascii = s.ToCharArray();
	String text = "";
	bool cont = true;
	while (cont)
	{
		for (int i = 0; i < ascii.Length; i++)
		{
			String tmp = text;
			tmp += ascii[i].ToString();
			String enc = Encode(tmp);
			String subStr = cipher.Substring(0, enc.Length);
			if (enc.Equals(subStr))
			{
				text = tmp;
				if (enc.Length == cipher.Length) cont = false;
				break;
			}

		}
	}
	Console.WriteLine(text);
}
```
=> Đáp án: `Sleeperio Sleeperio Disappeario Instanterio!`
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img10.png)
### => Flag: `corctf{4l1c3_15_1n_d33p_tr0ubl3_b3c4us3_1_d1d_n0t_s4v3_h3r!!:c}`

## zoom_zoom_vision
Tóm tắt chương trình: chương trình sẽ cho ta nhập một chuỗi (input) và biến đổi từng kí tự sang số sau đó in ra màn hình và so sánh với chuỗi số có trong chương trình.
Lợi dụng điều đó tôi giải quyết chall này một cách lươn lẹo, nhập "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!#@$%&*()^-|'?/\\[]<>.,_{}+=\`~" để lấy được kết quả, sử dụng kết quả đó và chuỗi số trong chương trình để tìm ra flag.

![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img11.png)

Code tìm flag:
```c
#include <iostream>
#include <string.h>
int num[100] = {1552, 1568, 1584, 1600, 1616, 1632, 1648, 1664, 1680, 
				1696, 1712, 1728, 1744, 1760, 1776, 1792, 1808, 1824, 
				1840, 1856, 1872, 1888, 1904, 1920, 1936, 1952, 1040, 
				1056, 1072, 1088, 1104, 1120, 1136, 1152, 1168, 1184, 
				1200, 1216, 1232, 1248, 1264, 1280, 1296, 1312, 1328, 
				1344, 1360, 1376, 1392, 1408, 1424, 1440, 768, 784, 
				800, 816, 832, 848, 864, 880, 896, 912, 528, 560, 
				1024, 576, 592, 608, 672, 640, 656, 1504, 720, 
				1984, 624, 1008, 752, 1472, 1456, 1488, 960, 
				992, 736, 704, 1520, 1968, 2000, 688, 976, 1536, 2016};
char text[] = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!#@$%&*()^-|'?/\\[]<>.,_{}+=`~";
int main ()
{
	int num2[50] = {1584, 1776, 1824, 1584, 1856, 1632, 1968, 1664, 768, 1728, 
				784, 784, 784, 784, 784, 1520, 1840, 1664, 784, 784, 784, 
				784, 784, 784, 816, 816, 816, 816, 816, 816, 1856, 1856, 
				1856, 1856, 1856, 1520, 784, 1856, 1952, 1520, 1584, 688, 688, 528, 2000};
	char tmp[2017] = {0};
	for (int j=0; j < strlen(text); j++) tmp[num[j]] = text[j];
	for (int i=0;i < 45; i++) printf("%c", tmp[num2[i]]);
	return 0;		
}
```

## => FLAG: `corctf{h0l11111_sh111111333333ttttt_1tz_c++!}`
