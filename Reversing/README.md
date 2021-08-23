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

Tiến hành debug và đặt breakpoint tại nơi nó đã load được từ resouces (cụ thể là hàn `button1_click`) và có thể thấy được `AliceInCeptiondream`
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img8.png)

Xem chi tiết `AliceInCeptiondream`:
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img9.png)

Sau một hồi phân tích hàm encode, tôi cảm thấy mất thời gian nên sẽ tiến hành sử dụng hàm encode để brute force lấy đáp án.
Code brute force:
```c#
class Program
    {
		static Program()
		{
			string[] array = new string[256];
			array[4] = "\u000f";
			array[5] = "\u0005\u0006\u0005\u0005\u0006";
			array[6] = "\u001d\u001d\u001d\u001d\u001d";
			array[7] = "\u0015\u0015\u0015\u0016\u0016";
			array[8] = "nmmnmn";
			array[9] = "ffff";
			array[10] = "~}}~";
			array[11] = "uvvu";
			array[12] = "\0";
			array[13] = "FFFF";
			array[14] = "^]]^";
			array[15] = "UVVU";
			array[36] = "\f\u000f\f\u000f\f\f";
			array[37] = "\u0004\a\u0004\u0004\a\u0004";
			array[38] = "\u001f\u001c\u001c\u001c\u001c";
			array[39] = "\u0014\u0014\u0014\u0014\u0017";
			array[40] = "ol";
			array[41] = "gg";
			array[42] = "||\u007f|";
			array[43] = "twtt";
			array[44] = "OL";
			array[45] = "GG";
			array[46] = "\\\\_\\";
			array[47] = "TWTT";
			array[68] = "\0";
			array[69] = "\0";
			array[70] = "\u001c\u001c\u001f\u001f\u001f";
			array[71] = "\u0017\u0017\u0017\u0014\u0014\u0014";
			array[72] = "olll";
			array[73] = "dggg";
			array[74] = "|\u007f|";
			array[75] = "wwtt";
			array[76] = "OLLL";
			array[77] = "DGGG";
			array[78] = "\\_\\";
			array[79] = "WWTT";
			array[100] = "\0";
			array[101] = "\u0005\u0006\u0005\u0006\u0005";
			array[102] = "\u001d\u001d\u001d\u001e\u001e";
			array[103] = "\u0016\u0015\u0016\u0015\u0016\u0015";
			array[104] = "nmnm";
			array[105] = "fef";
			array[106] = "}}}";
			array[107] = "\0";
			array[108] = "NMNM";
			array[109] = "FEF";
			array[110] = "]]]";
			array[111] = "\0";
			array[132] = "\n\n\n\n\n";
			array[133] = "\u0001\u0001\u0002\u0002\u0001\u0001";
			array[134] = "\u001a\u001a\u001a\u001a\u0019";
			array[135] = "\0";
			array[136] = "ijj";
			array[137] = "babb";
			array[138] = "y";
			array[139] = "\0";
			array[140] = "IJJ";
			array[141] = "BABB";
			array[142] = "Y";
			array[143] = "\0";
			array[164] = "\0";
			array[165] = "\0\u0003\u0003\u0003\u0003\0";
			array[166] = "\u001b\u001b\u001b\u001b\u001b";
			array[167] = "\u0010\u0013\u0013\u0013\u0010";
			array[168] = "k";
			array[169] = "``";
			array[170] = "{{x";
			array[171] = "\0";
			array[172] = "K";
			array[173] = "@@";
			array[174] = "[[X";
			array[175] = "\0";
			array[196] = "\b\v\b\b\b";
			array[197] = "\0\u0003\0\u0003\0\u0003";
			array[198] = "\u001b\u0018\u0018\u0018\u0018";
			array[199] = "\0";
			array[200] = "hhkh";
			array[201] = "c`";
			array[202] = "xxx{";
			array[203] = "\0";
			array[204] = "HHKH";
			array[205] = "C@";
			array[206] = "XXX[";
			array[207] = "\0";
			array[228] = "\t\n\n\n\n\t";
			array[229] = "\u0002\u0001\u0001\u0002\u0001";
			array[230] = "\u001a\u001a\u0019\u0019\u0019";
			array[231] = "\u0011\u0011\u0012\u0012\u0011\u0011";
			array[232] = "jji";
			array[233] = "bbb";
			array[234] = "yzz";
			array[235] = "qqrrqr";
			array[236] = "JJI";
			array[237] = "BBB";
			array[238] = "YZZ";
			rm = array;
			xm = 1056017893861212352UL;
		}
		private static readonly string[] rm;
		private static readonly ulong xm;
		
		static void Main(string[] args)
		{
			String s = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789! ";
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

		[DllImport("kernel32.dll", ExactSpelling = true, SetLastError = true)]
		private static extern bool CheckRemoteDebuggerPresent(IntPtr hProcess, ref bool isDebuggerPresent);
		
		public static string Encode(string plaintext)
		{
			string text = string.Empty;
			if (!string.IsNullOrEmpty(plaintext))
			{
				string text2 = string.Join("/", plaintext.Select(new Func<char, string>(dF)).ToArray<string>());
				byte[] array = new byte[]
				{
					(byte)(xm & 255UL),
					(byte)(xm >> 8 & 255UL),
					(byte)(xm >> 16 & 255UL),
					(byte)(xm >> 24 & 255UL),
					(byte)(xm >> 32 & 255UL),
					(byte)(xm >> 40 & 255UL),
					(byte)(xm >> 48 & 255UL),
					(byte)(xm >> 56 & 255UL)
				};		
				for (int i = 0; i < text2.Length; i++)
				{
					text += string.Format("{0:x2}", (byte)text2[i] ^ array[i % array.Length]);
				}
				
			}
			bool flag = false;
			if (!CheckRemoteDebuggerPresent(Process.GetCurrentProcess().Handle, ref flag) && flag)
			{
				text = string.Join<char>("", text.Reverse<char>().ToArray<char>());
			}
			return text;
		}
		private static byte ror(byte v, byte s)
        {
            byte b = (byte)(s % 8);
            return (byte)((int)v << (int)b | v >> (int)(8 - b));
        }
        private static byte rol(byte v, byte s)
        {
            byte b = (byte)(s % 8);
            return (byte)(v >> (int)b | (int)v << (int)(8 - b));
        }
		

		[CompilerGenerated]
		internal static char a(char c)
		{
			int n = rol((byte)c, 3);
			return (char) n;
		}

		[CompilerGenerated]
		internal static char b(char c)
		{
			int n = ror((byte)c, 5);
			return (char) n;
		}
		[CompilerGenerated]
		internal static string cF(char c, string x)
		{
			String s = new string((from y in x
								   select (char)((byte)y ^ (byte)c)).ToArray<char>());
			return s;
		}
		[CompilerGenerated]
		internal static string dF(char c)
		{
			int n = (int)a(c);
			String tmp = rm[n];
			String s = string.Join("", new string[]
			{
				cF(c, new string (tmp.Select(new Func<char, char>(b)).ToArray<char>()))
			});
			return s;
		}
}
```
=> Đáp án: `Sleeperio Sleeperio Disappeario Instanterio!`
![alt text](https://github.com/konoha279/write-up-corCTF/blob/rev/Reversing/Resources/img10.png)
### => Flag: `corctf{4l1c3_15_1n_d33p_tr0ubl3_b3c4us3_1_d1d_n0t_s4v3_h3r!!:c}`