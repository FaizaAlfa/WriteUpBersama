# interencdec #
 
## Overview ##

Author: NGIRIMANA Schadrack

Category: [Cryptography](../)

## Description ##

> Can you get the real meaning from this file.

> Download the file [here](files/enc_flag).

## Approach ##

Diberikan file enc_flag yang diakhiri dengan == yang menandakan itu adalah base64, setelah itu saya langsung menggunakan tools bernama [cyberchef](https://gchq.github.io/CyberChef/) kemudian gunakan from base64 dan mendapatkan hasil:

```
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2xoNjBsMDBpfQ=='
```

dan merupakan base64 lagi kemudian kita coba masukkan lagi dan mendapatkan hasil:

```
wpjvJAM{jhlzhy_k3jy9wa3k_lh60l00i}
```

yang sepertinya huruf yang digeser atau caesar cipher, coba di cari pergeseran berapa dan menemukan di 19 

### Flag: `picoCTF{caesar_d3cr9pt3d_ea60e00b}`