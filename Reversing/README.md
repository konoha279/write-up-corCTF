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
=> Flag: `corctf{see?_rEv_aint_so_bad}`