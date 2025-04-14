general template
```
title "display word in center of screen and also the count from a string input"

.model small
.stack 100h

.data
    enter 0ah, 0dh, "$"
    input db 100, ?, 100 dup('$')
    wordBuffer db 100, ?, 100 dup('$')
    strlen db ?
    
.code
main proc
    end mainp
    
clearscreen proc
    mov ah, 00
    mov al, 03
    int 10h
    ret
    clearscreen endp

display proc
    mov ah, 09h
    int 21h
    ret
    display endp

newline proc
    lea dx, enter
    call display
    ret
    newline endp

displayNumber proc
    push ax
    push bx
    push cx
    push dx
    
    mov cx, 0
    convert_loop:
    mov dx, 0
    mov bx, 10
    div bx
    push dx         
    inc cx
    cmp ax, 0
    jne convert_loop
    
    display_loop:
    pop dx 
    add dl, '0'
    mov ah, 02h
    int 21h    
    loop display_loop
    
    pop dx
    pop cx
    pop bx
    pop ax
    ret
    displayNumber endp
setcursor proc
	;mov dh, 00h; row
	;mov dl, 00h;column
	; ch and cl can be used for start and end of scan line
	mov ah, 02h
	mov bh, 00h
	int 10h
	ret
	setcursor endp
	
```

basic input new line and also displaying it
```
.model small
.stack 100h
.data 
buffer db 100
       db ?
       db 100 dup('$')
.code
main proc 
	mov ax, @data
	mov ds, ax
	
	mov ah, 0ah
	lea dx, buffer
	int 21h
	
	
	;display new line
	mov dx, 0ah
	mov ah, 02h
	int 21h
	mov dl, 0Dh
	int 21h
	
	;display input string
	mov dx, offset buffer+2
	mov ah, 09h
	int 21h
	
	
	;exit program
	mov ah, 4ch
	int 21h
	main endp
```


max characters
```
buffer DB 255, ?, 255 DUP('$')  ; Max 255 chars input (DOS limit)
buffer DW 50 DUP(0)  ; Can store 50 words = 100 characters
buffer DD 100 DUP(0)  ; Can store 400 characters
buffer DQ 50 DUP(0)  ; Can store 400 characters


```

For standard DOS string input (`INT 21h`), always use **`DB`** because it handles character-by-character storage.

function to clear screen
```asm
clearscreen proc
mov ah, 00
mov al, 03
int 10h
ret
clearscreen endp

;alternatively
clearscreen proc
mov ax, 0600h
mov bh, 07h
mov cx, 0000
mov dx, 184fh
int 10h
ret
clearscreen endp
```

function to display new line
```
newline proc
mov dx, 0ah
mov ah, 02h
int 21h
mov dl, 0Dh
int 21h
ret
newline endp

;alternatively
.data
newlineString db 0ah, 0dh, "$"
newline proc
lea dx, newlineString
call display; display function should be defined.
ret
newline endp
```

to display string
```
; lea dx, variableName
display proc
	mov ah, 09h
	int 21h
ret
display endp
```

to enter characters without echo and display it
```
title "program to enter characters from keyboard"
.model small
.stack 100

.data
char_buf db 20 dub(?)

.code 
    mov ax, @data
    mov ds, ax
    lea si, char_buf
    mov ah, 07 ; putting ah= 01h gives character echo too 
    again: int 21h
    mov [si], al
    inc si
    cmp al, 0dh
    jne again     
    lea di, char_buf
    mov ah, 02
    back: mov dl, [di]
    int 21h
    inc di
    cmp byte ptr [di], 0dh
    jne back
    mov ax, 4c00h
    int 21h
end
```

to set cursor
```
setcursor proc
mov dh, 00h; row
mov dl, 00h;column ch and cl can be used for start and end of scan line
mov ah, 02h
mov bh, 00h
int 10h
ret
setcursor endp
```

to reverse string
```
;----------------------------------------------------------
; REVERSE: Reverse the processed string (in place).
; The string to be reversed starts at string+2 and ends at the
; '$' terminator.
;----------------------------------------------------------
reverse proc
    lea si, string+2        ; SI points to the beginning
    ; Find the end of the string (the '$' character)
    mov di, si
find_end:
    cmp byte ptr [di], '$'
    je done_find
    inc di
    jmp find_end
done_find:
    dec di                  ; DI now points to last character (not '$')
reverse_loop:
    cmp si, di
    jge reverse_done        ; When pointers meet or cross, we are done
    ; Swap the characters at SI and DI
    mov al, [si]
    mov bl, [di]
    mov [si], bl
    mov [di], al
    inc si
    dec di
    jmp reverse_loop
reverse_done:
    ret
reverse endp

```

two digit number conversion to ascii
```
; Converts numeric count to ASCII string
convert_upper_to_ascii proc
    mov al, upper
    aam                   ; Divide AL by 10
    add ax, '00'          ; Convert to ASCII
    ; Handle single-digit case
    cmp ah, '0'
    jne two_digits
    mov count_str[0], al
    mov count_str[1], '$'
    jmp exit
two_digits:
    mov count_str[0], ah  ; Tens digit
    mov count_str[1], al  ; Ones digit
    mov count_str[2], '$'
exit:
    ret
convert_upper_to_ascii endp

```

convert number
```
.data
displayBuffer 100 dup('$')


display_number proc
mov ax, 0 
mov al, [vowelcount] 
mov bl, 10 
div bl 
add al, '0' 
add ah, '0' 
mov displayBuffer, 
al mov displayBuffer+1, ah
```

display multiple number
```
title "Count the Number of Vowels"
.model small
.stack 100h

.data
    newlineString db 0ah, 0dh, "$"      
    num dw 10000
    num1 db 100
    numBuffer db 6 dup('$')   ; Buffer for storing the ASCII result

.code
main proc
    mov ax, @data
    mov ds, ax 

    ; Pass the actual value, not the address    
    mov ax, num
    call display_number   
    call newline
    
    mov ax, 0000
    mov al, num1
    call display_number
    
    mov ah, 4ch
    int 21h
main endp
 
clearscreen proc 
    mov ah, 00h
    mov al, 03h
    int 10h  
    ret
clearscreen endp   

display proc 
    mov ah, 09h
    int 21h
    ret
display endp  

newline proc
    lea dx, newlineString
    call display
    ret
newline endp  
 
display_number proc
    push ax
    push bx
    push cx
    push dx

    ; Convert AX to ASCII and display
    mov cx, 0              ; Counter for digits
convert_loop:
    mov dx, 0
    mov bx, 10
    div bx                  ; AX = AX / 10, DX = AX % 10
    push dx                 ; Save remainder (digit)
    inc cx                  ; Increment digit count
    cmp ax, 0               ; If AX == 0, we're done
    jne convert_loop

display_loop:
    pop dx                  ; Get digit from stack
    add dl, '0'             ; Convert to ASCII
    mov ah, 02h             ; DOS display character function
    int 21h
    loop display_loop       ; Repeat for all digits

    pop dx
    pop cx
    pop bx
    pop ax
    ret
display_number endp
 
end main 

```

display word at the center:
```
center proc
;   It computes the left margin as (80 - word_length) / 2 and prints
    push ax
    push bx
    push cx
    
    mov cx, di ; di holds word length
    mov ax, 80
    sub ax, cx
    shr ax, 1
    mov bx, ax
    
    printspaces:
    cmp bx, 0
    je printword
    lea dx, space
    call display
    dec bx
    jmp printspaces
    
    printword:
    lea dx, wordBuffer
    call display
    
    
    pop cx
    pop bx
    pop ax
    ret
    center endp
```

displayIndividualWord
```
displayIndividualWord proc
    lea si, input+2
    mov cl, [input+1]
    mov ch, 0
    mov di, 0
    
    process_loop:
    cmp cx, 0
    je finish
    dec cx
    lodsb
    cmp al, 0dh
    je displayword
    cmp al, ' '
    je displayword
    
    mov [wordBuffer+di], al
    inc di
    jmp process_loop
    
    displayword:
    cmp di, 0
    je skip
    mov [wordBuffer+di], '$'
    call center 
    
    mov di, 0
    call newline
    skip: 
    jmp process_loop 
    
    finish: cmp di, 0
    je done
    mov [wordBuffer+di], '$'
    call center
    call newline     
    
    done: ret
    
    displayIndividualWord endp
```

reversed string
```
 reversed proc
    mov cl, input[1]
    mov ch, 0
    lea si, input+2
    add si, cx
    dec si
    lea di, output
    
    next: mov al, [si]
    mov [di], al
    dec si
    inc di
    loop next
    
    
    ret
    reversed endp
```

password
```
authentication proc  
    lea si, input
    lea di, password 
    mov cx, 20
    next: mov ah, 07h
    int 21h
    cmp al, 0dh
    je check 
    mov [si], al 
    inc si
    loop next 
    check: lea si, input
    check_loop: mov al, [di] 
    cmp al, '$'
    je match
    cmp al, [si]
    jne error
    inc si
    inc di  
    jmp check_loop
    error: call clearscreen
    lea dx, msg3
    call display
    jmp exit
    match:
    call clearscreen
    lea dx, msg2
    call display
    exit: ret    
    authentication endp
```

lowercase
```
lowercase proc 
    lea si, input 
    l1: mov al, [si]
    cmp al, '$'  ; End of word marker
    je l2
    cmp al, 'A'
    jb after
    cmp al, 'Z'
    jbe convert   ; Convert to lowercase if it's an uppercase letter
    
    jmp after
    
    convert: add al, 20h  
    mov [si], al; Convert uppercase to lowercase
    after: inc si
    jmp l1
    l2: call clearscreen
    lea dx, input
    call display  ; Display the word in lowercase 
    ret
    lowercase endp
end main
```

firstUpperAllLower
```
lowercaseFirstUpper proc 
    lea si, input 
    l1: mov al, [si]
    cmp al, '$'  ; End of word marker
    je l2
    cmp al, 'A'
    jb after
    cmp al, 'Z'
    jbe convert   ; Convert to lowercase if it's an uppercase letter
    
    jmp after
    
    convert: add al, 20h  
    mov [si], al; Convert uppercase to lowercase
    after: inc si
    jmp l1
    l2:
    call firstUpper
    ret
    lowercaseFirstUpper endp 

firstUpper proc
    lea si, input
    mov al, [si]
    cmp al, '$'
    je l4
    sub al, 20h 
    mov [si], al
    l4: lea dx, input
    call display
    ret
    firstUpper endp
end main        
```