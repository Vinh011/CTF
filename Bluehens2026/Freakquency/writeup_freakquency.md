# CTF Write-Up: Freakquency
---

## Đề bài

<img width="620" height="494" alt="image" src="https://github.com/user-attachments/assets/5bf840e4-05e1-40c6-ae73-7a1bc9afd489" />

```
You have the audacity to match my freak?
```

**File đính kèm:** `hidden_message.wav`

---

Mở file âm thanh và thấy gì đó lạ lạ nên tôi zoom nó lên và nó là mã base64:

<img width="1919" height="866" alt="image" src="https://github.com/user-attachments/assets/8256dca5-e33c-4452-8d69-95a470769a91" />


`VURDVEZ7dzB3X3kwdV9jNG5faDM0cl80X2YxbDM/f0==`

Decode nó ra và đc flag:

```
UDCTF{w0w_y0u_c4n_h34r_4_f1l3?}
```
