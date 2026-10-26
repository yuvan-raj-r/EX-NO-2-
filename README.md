## EX. NO:2 IMPLEMENTATION OF PLAYFAIR CIPHER

 

## AIM:

To write a C program to implement the Playfair Substitution technique.

## DESCRIPTION:

The Playfair cipher starts with creating a key table. The key table is a 5×5 grid of letters that will act as the key for encrypting your plaintext. Each of the 25 letters must be unique and one letter of the alphabet is omitted from the table (as there are 25 spots and 26 letters in the alphabet).

To encrypt a message, one would break the message into digrams (groups of 2 letters) such that, for example, "HelloWorld" becomes "HE LL OW OR LD", and map them out on the key table. The two letters of the diagram are considered as the opposite corners of a rectangle in the key table. Note the relative position of the corners of this rectangle. Then apply the following 4 rules, in order, to each pair of letters in the plaintext:
1.	If both letters are the same (or only one letter is left), add an "X" after the first letter
2.	If the letters appear on the same row of your table, replace them with the letters to their immediate right respectively
3.	If the letters appear on the same column of your table, replace them with the letters immediately below respectively
4.	If the letters are not on the same row or column, replace them with the letters on the same row respectively but at the other pair of corners of the rectangle defined by the original pair.
## EXAMPLE:
![image](https://github.com/Hemamanigandan/EX-NO-2-/assets/149653568/e6858d4f-b122-42ba-acdb-db18ec2e9675)

 

## ALGORITHM:

STEP-1: Read the plain text from the user.
STEP-2: Read the keyword from the user.
STEP-3: Arrange the keyword without duplicates in a 5*5 matrix in the row order and fill the remaining cells with missed out letters in alphabetical order. Note that ‘i’ and ‘j’ takes the same cell.
STEP-4: Group the plain text in pairs and match the corresponding corner letters by forming a rectangular grid.
STEP-5: Display the obtained cipher text.

Program:
```
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define SIZE 100
#define BUF 202   /* room for inserted filler letters */

/* Keep letters only, lowercase them, treat j as i. Returns new length. */
int cleanText(char *s)
{
    int i, count = 0;
    char c;
    for (i = 0; s[i] != '\0'; i++) {
        if (isalpha((unsigned char)s[i])) {
            c = (char)tolower((unsigned char)s[i]);
            if (c == 'j') c = 'i';
            s[count++] = c;
        }
    }
    s[count] = '\0';
    return count;
}

/* Build the 5x5 key square (no 'j') */
void generateKeyTable(const char key[], int ks, char keyT[5][5])
{
    int used[26] = {0};
    int pos = 0, k, idx;

    for (k = 0; k < ks; k++) {
        idx = key[k] - 'a';
        if (!used[idx]) {
            used[idx] = 1;
            keyT[pos / 5][pos % 5] = key[k];
            pos++;
        }
    }
    for (k = 0; k < 26; k++) {
        if (k == 'j' - 'a') continue;
        if (!used[k]) {
            keyT[pos / 5][pos % 5] = (char)('a' + k);
            pos++;
        }
    }
}

/* Find row/col of both letters of a digraph */
void search(char keyT[5][5], char a, char b, int arr[])
{
    int i, j;
    for (i = 0; i < 5; i++) {
        for (j = 0; j < 5; j++) {
            if (keyT[i][j] == a) { arr[0] = i; arr[1] = j; }
            if (keyT[i][j] == b) { arr[2] = i; arr[3] = j; }
        }
    }
}

int mod5(int a)
{
    return a % 5;
}

/* Insert filler between repeated letters, pad a lone last letter. */
int prepare(char str[], int len)
{
    char tmp[BUF];
    char a, filler;
    int n = 0, i = 0;

    while (i < len) {
        a = str[i];
        filler = (a == 'x') ? 'q' : 'x';
        tmp[n++] = a;
        if (i + 1 >= len) {
            tmp[n++] = (a == 'z') ? 'x' : 'z';
            i++;
        } else if (a == str[i + 1]) {
            tmp[n++] = filler;
            i++;
        } else {
            tmp[n++] = str[i + 1];
            i += 2;
        }
    }
    tmp[n] = '\0';
    strcpy(str, tmp);
    return n;
}

void encrypt(char str[], char keyT[5][5], int ps)
{
    int i, a[4];

    for (i = 0; i < ps; i += 2) {
        search(keyT, str[i], str[i + 1], a);

        if (a[0] == a[2]) {            /* same row */
            str[i]     = keyT[a[0]][mod5(a[1] + 1)];
            str[i + 1] = keyT[a[0]][mod5(a[3] + 1)];
        } else if (a[1] == a[3]) {     /* same column */
            str[i]     = keyT[mod5(a[0] + 1)][a[1]];
            str[i + 1] = keyT[mod5(a[2] + 1)][a[1]];
        } else {                       /* rectangle */
            str[i]     = keyT[a[0]][a[3]];
            str[i + 1] = keyT[a[2]][a[1]];
        }
    }
}

/* str must have room for BUF chars */
void encryptByPlayfairCipher(char str[], char key[])
{
    char keyT[5][5];
    int ks, ps;

    ks = cleanText(key);
    ps = cleanText(str);
    ps = prepare(str, ps);
    generateKeyTable(key, ks, keyT);
    encrypt(str, keyT, ps);
}

/* Read a line safely and strip the newline. Returns 1 on success. */
int readLine(const char *prompt, char *buf, int size)
{
    printf("%s", prompt);
    if (fgets(buf, size, stdin) == NULL) return 0;
    buf[strcspn(buf, "\n")] = '\0';
    return 1;
}

int hasLetter(const char *s)
{
    int i;
    for (i = 0; s[i] != '\0'; i++)
        if (isalpha((unsigned char)s[i])) return 1;
    return 0;
}

int main(void)
{
    char str[BUF], key[SIZE];

    if (!readLine("Enter the key: ", key, SIZE)) {
        printf("Failed to read key.\n");
        return 1;
    }
    if (!readLine("Enter the plain text: ", str, SIZE)) {
        printf("Failed to read plain text.\n");
        return 1;
    }

    if (!hasLetter(key) || !hasLetter(str)) {
        printf("Key and plain text must each contain at least one letter.\n");
        return 1;
    }

    printf("\nKey text: %s\n", key);
    printf("Plain text: %s\n", str);

    encryptByPlayfairCipher(str, key);
    printf("Cipher text: %s\n", str);

    return 0;
}
```





## Output:
<img width="1686" height="822" alt="image" src="https://github.com/user-attachments/assets/2ecae9aa-beb7-404d-8de1-2be6bbf0aaef" />



## RESULT:
The program is executed successfully



