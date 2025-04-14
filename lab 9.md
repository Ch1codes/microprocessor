q1
```
; ========================================================================
; 8086 Assembly Program to count the number of alphabets, numerals,
; and other characters from user input and display the results.
;
; The program:
; 1. Accepts a string from the user (max 100 characters).
; 2. Counts the number of alphabets (A-Z, a-z), numerals (0-9),
;    and other characters (special symbols, spaces, etc.).
; 3. Displays the counts.
; ========================================================================

.MODEL SMALL          ; Use small memory model (single segment for code and data)
.STACK 100H          ; Define stack size (256 bytes)

.DATA
    ; =================== INPUT BUFFER ===================
    buffer DB 100, ?, 100 DUP('$')   ; Buffer for storing user input
    ; First byte (100) - Maximum input size
    ; Second byte (?) - Stores actual input length after input is taken
    ; Remaining 100 bytes - Stores the input string

    ; ================== CHARACTER COUNTERS ==================
    alphabets DB 0     ; Count of alphabetic characters
    other DB 0         ; Count of other characters (symbols, spaces, etc.)
    numeral DB 0       ; Count of numeric characters (0-9)

    ; ================== DISPLAY MESSAGES ==================
    msg1 DB "No. of numerals: $"
    msg2 DB "No. of alphabets: $"
    msg3 DB "No. of other characters: $"

    ; Newline sequence for better formatting when displaying results
    newlineStr DB 0Dh, 0Ah, "$"    

    ; Buffer to store the converted number as an ASCII string
    numStr DB "  ", "$"    

.CODE
MAIN PROC
    ; =================== INITIALIZE DATA SEGMENT ===================
    MOV AX, @DATA    ; Load data segment address
    MOV DS, AX       ; Set DS to the data segment

    CALL ClearScreen  ; Clear the screen before execution

    ; =================== TAKE USER INPUT ===================
    MOV AH, 0Ah       ; DOS function 0Ah: Buffered input
    LEA DX, buffer    ; Load the address of the input buffer
    INT 21H           ; Call DOS interrupt to take input

    ; =================== SET INPUT LENGTH ===================
    MOV CL, buffer[1]  ; The second byte of buffer contains actual input length
    MOV CH, 0          ; Clear CH, making CX a valid loop counter
    JCXZ SkipCount     ; If CX is zero (no input), skip the counting process

    ; =================== START CHARACTER COUNTING ===================
    LEA SI, buffer + 2  ; Start processing input from the third byte (actual data start)

CountLoop:
    MOV AL, [SI]        ; Load the current character from input string

    ; ========== CHECK IF THE CHARACTER IS A NUMERAL (0-9) ==========
    CMP AL, '0'         ; Compare with ASCII '0'
    JB CheckAlphabet    ; If below '0', check if it's an alphabet
    CMP AL, '9'         ; Compare with ASCII '9'
    JBE IsNumber        ; If between '0' and '9', it's a number

CheckAlphabet:
    ; ========== CHECK IF THE CHARACTER IS AN UPPERCASE LETTER (A-Z) ==========
    CMP AL, 'A'         ; Compare with 'A'
    JB CheckLowerCase   ; If below 'A', check for lowercase
    CMP AL, 'Z'         ; Compare with 'Z'
    JBE IsAlphabet      ; If between 'A' and 'Z', it's an alphabet

CheckLowerCase:
    ; ========== CHECK IF THE CHARACTER IS A LOWERCASE LETTER (a-z) ==========
    CMP AL, 'a'         ; Compare with 'a'
    JB IsOther          ; If below 'a', it is neither alphabet nor number
    CMP AL, 'z'         ; Compare with 'z'
    JBE IsAlphabet      ; If between 'a' and 'z', it's an alphabet

IsNumber:
    INC numeral         ; Increment numeral count
    JMP NextChar        ; Move to next character

IsAlphabet:
    INC alphabets       ; Increment alphabet count
    JMP NextChar        ; Move to next character

IsOther:
    INC other           ; Increment count of other characters

NextChar:
    INC SI              ; Move to the next character
    LOOP CountLoop      ; Decrement CX and repeat loop if CX is not zero

SkipCount:

    ; =================== DISPLAY RESULTS ===================

    ; Print numerals count
    CALL NewLine
    LEA DX, msg1        ; Load address of "No. of numerals: "
    CALL DisplayString
    MOV AL, numeral     ; Move numeral count to AL for printing
    CALL PrintNumber

    ; Print alphabets count
    CALL NewLine
    LEA DX, msg2        ; Load address of "No. of alphabets: "
    CALL DisplayString
    MOV AL, alphabets   ; Move alphabet count to AL for printing
    CALL PrintNumber

    ; Print other characters count
    CALL NewLine
    LEA DX, msg3        ; Load address of "No. of other characters: "
    CALL DisplayString
    MOV AL, other       ; Move other characters count to AL for printing
    CALL PrintNumber

    ; =================== TERMINATE PROGRAM ===================
    MOV AH, 4Ch        ; DOS function 4Ch: Terminate program
    INT 21H            ; Call DOS to exit

MAIN ENDP

; =================== PROCEDURES ===================

; ---------- PROCEDURE: CLEAR SCREEN ----------
ClearScreen PROC
    MOV AH, 00h        ; BIOS function 00h: Set video mode
    MOV AL, 03h        ; Set video mode to 03h (text mode, 80x25)
    INT 10h            ; Call BIOS interrupt 10h
    RET
ClearScreen ENDP

; ---------- PROCEDURE: NEW LINE ----------
NewLine PROC
    LEA DX, newlineStr  ; Load address of newline string (CRLF)
    MOV AH, 09h         ; DOS function 09h: Print string
    INT 21H             ; Call DOS interrupt
    RET
NewLine ENDP

; ---------- PROCEDURE: DISPLAY STRING ----------
DisplayString PROC
    MOV AH, 09h         ; DOS function 09h: Print string
    INT 21H             ; Call DOS interrupt
    RET
DisplayString ENDP

; ---------- PROCEDURE: PRINT NUMBER ----------
PrintNumber PROC
    ; Convert single-digit number in AL to ASCII and display it
    AAM                 ; Convert AL (binary) to ASCII (AH = tens, AL = ones)
    ADD AX, 3030h       ; Convert binary digits to ASCII ('0' = 30h)
    MOV numStr[0], AH   ; Store first ASCII digit in numStr
    MOV numStr[1], AL   ; Store second ASCII digit in numStr
    LEA DX, numStr      ; Load address of numStr into DX
    CALL DisplayString  ; Call DisplayString to print it
    RET
PrintNumber ENDP

; =================== END OF PROGRAM ===================
END MAIN


```


q2
```
; 8086 Assembly Program to compute the sum of the series:
;   (1+x)*5 + (3+x)*6 + (5+x)*7 + ... for 10 terms,
; where x is a single digit provided by the user.
;
; The program uses DOS interrupts for input/output.
; It first prompts the user for a digit, then computes the sum,
; converts the numeric sum into an ASCII string, and finally displays it.

.MODEL small
.STACK 100h

.DATA
; Prompt message asking the user to enter a single digit.
msg1    DB 'Enter a single digit number: $'

; Message that will precede the display of the computed sum.
msg2    DB 0Dh,0Ah, 'The sum is: $'    ; 0Dh,0Ah represent CR (carriage return) and LF (line feed).

; Variable to store the user-provided digit (after converting from ASCII).
xval    DB ?             

; Buffer to hold the converted sum as an ASCII string.
; We reserve 6 bytes (enough for up to 5 digits plus a terminating '$').
buffer  DB 6 DUP(?)      

; Constant for the number of terms in the series (10 terms).
ten     DW 10           

.CODE
main    PROC
        ;-----------------------------------------------------
        ; Set up the data segment.
        ;-----------------------------------------------------
        MOV AX, @DATA    ; Load the address of the data segment into AX.
        MOV DS, AX       ; Initialize the DS register with the data segment address.

        ;-----------------------------------------------------
        ; Prompt the user to input a single digit.
        ;-----------------------------------------------------
        LEA DX, msg1     ; Load the effective address of msg1 into DX.
        MOV AH, 09h      ; DOS function 09h: Display string until '$' is encountered.
        INT 21h          ; Call DOS interrupt to display the prompt.

        ;-----------------------------------------------------
        ; Read a single character from the keyboard.
        ;-----------------------------------------------------
        MOV AH, 01h      ; DOS function 01h: Read character from standard input (echoes it).
        INT 21h          ; The character is returned in AL.
        SUB AL, '0'      ; Convert ASCII code for digit into its numeric value (e.g., '5' becomes 5).
        MOV xval, AL     ; Store the numeric value in xval.

        ;-----------------------------------------------------
        ; Initialize loop variables for calculating the sum.
        ;-----------------------------------------------------
        MOV CX, 10       ; Set loop counter to 10 (for 10 terms in the series).
        XOR SI, SI       ; Clear SI to 0; SI will be used as the loop index (i).
        XOR DI, DI       ; Clear DI to 0; DI will accumulate the running sum.

series_loop:
        ;-----------------------------------------------------
        ; Compute the first factor for the current term: (2*i + 1 + x)
        ;-----------------------------------------------------
        MOV AX, SI       ; Move the current index (i) into AX.
        SHL AX, 1        ; Multiply i by 2 (AX = 2*i).
        INC AX           ; Add 1 (AX = 2*i + 1).
        MOV DL, xval     ; Load the user input (x) into DL.
        MOV DH, 0        ; Clear DH to form a full 16-bit value in DX.
        ADD AX, DX       ; Add x to AX. Now, AX = (2*i + 1 + x).

        ;-----------------------------------------------------
        ; Compute the second factor for the current term: (i + 5)
        ;-----------------------------------------------------
        MOV BX, SI       ; Copy the current index (i) into BX.
        ADD BX, 5        ; Add 5 to BX. Now, BX = (i + 5).

        ;-----------------------------------------------------
        ; Multiply the two factors.
        ;-----------------------------------------------------
        MUL BX           ; Multiply AX by BX.
                         ; The result is stored in DX:AX.
                         ; (We assume the result fits in AX, so DX remains 0.)

        ;-----------------------------------------------------
        ; Add the product of the factors to the running sum.
        ;-----------------------------------------------------
        ADD DI, AX       ; DI = DI + (product of (2*i + 1 + x) and (i + 5)).

        ;-----------------------------------------------------
        ; Increment loop index and repeat for next term.
        ;-----------------------------------------------------
        INC SI         ; Increment the loop index i.
        LOOP series_loop ; Decrement CX and loop if CX is not zero.

        ;-----------------------------------------------------
        ; At this point, DI holds the total sum.
        ; Convert the numeric sum in DI to an ASCII string.
        ;-----------------------------------------------------
        MOV AX, DI       ; Move the sum into AX (the input for ConvertToString).
        CALL ConvertToString   ; Call the subroutine to convert AX to a string in 'buffer'.

        ;-----------------------------------------------------
        ; Display the result message and the computed sum.
        ;-----------------------------------------------------
        LEA DX, msg2     ; Load the effective address of msg2 into DX.
        MOV AH, 09h      ; DOS function 09h: Display string.
        INT 21h          ; Call DOS interrupt to display the message.

        LEA DX, buffer   ; Load the effective address of the converted sum (buffer) into DX.
        MOV AH, 09h      ; DOS function 09h: Display string.
        INT 21h          ; Call DOS interrupt to display the sum.

        ;-----------------------------------------------------
        ; Terminate the program.
        ;-----------------------------------------------------
        MOV AH, 4Ch      ; DOS function 4Ch: Terminate process.
        INT 21h          ; Call DOS interrupt to exit.
main    ENDP

;-------------------------------------------------
; ConvertToString Subroutine:
;   Converts an unsigned integer in AX into its ASCII decimal string.
;   The result is stored in the global variable 'buffer' and is terminated
;   with a '$' character, as required by DOS function 09h.
;
; Input:
;   AX = Unsigned integer to convert.
;
; Output:
;   buffer = ASCII string representation of the number (terminated with '$').
;-------------------------------------------------
ConvertToString PROC
        ;-------------------------------------------------
        ; Save registers that will be modified.
        ;-------------------------------------------------
        PUSH BX          ; Save BX.
        PUSH CX          ; Save CX.
        PUSH DX          ; Save DX.
        PUSH SI          ; Save SI.

        ;-------------------------------------------------
        ; Initialize a digit counter.
        ;-------------------------------------------------
        MOV CX, 0        ; Set CX = 0; this will count the number of digits processed.

        ;-------------------------------------------------
        ; Check if the number in AX is zero.
        ;-------------------------------------------------
        CMP AX, 0        ; Compare AX with 0.
        JNE ConvertLoop  ; If AX is not zero, proceed to the conversion loop.
        
        ; Special case: if the number is 0, directly store '0' and terminate.
        MOV BYTE PTR buffer, '0'  ; Store ASCII '0' in the first byte of buffer.
        MOV BYTE PTR buffer+1, '$' ; Append the '$' terminator.
        JMP ConvertDone           ; Skip the conversion loop.

ConvertLoop:
        ;-------------------------------------------------
        ; Divide the number in AX by 10 to extract a digit.
        ;-------------------------------------------------
        MOV BX, 10       ; Set BX to 10 (divisor).
        XOR DX, DX       ; Clear DX to prepare for division (DX:AX is the dividend).
        DIV BX           ; Divide DX:AX by 10.
                         ; After DIV: AX contains the quotient, DX contains the remainder.
                         ; The remainder is the current digit (least significant digit).
        PUSH DX          ; Push the remainder (digit) onto the stack.
        INC CX           ; Increment the digit counter.
        CMP AX, 0        ; Check if the quotient is zero.
        JNE ConvertLoop  ; If not, continue dividing the quotient by 10.

        ;-------------------------------------------------
        ; Now all digits are stored on the stack in reverse order.
        ; Pop them and store in the buffer in correct order.
        ;-------------------------------------------------
        LEA SI, buffer   ; Load the starting address of buffer into SI.
ConvertPop:
        POP DX           ; Pop a digit from the stack (digit in DX, only DL is used).
        ADD DL, '0'      ; Convert the numeric digit to its ASCII equivalent by adding the ASCII code for '0'.
        MOV [SI], DL     ; Store the ASCII character in the current buffer location.
        INC SI           ; Move to the next character position in buffer.
        LOOP ConvertPop  ; Decrement CX and loop until all digits have been processed.

        ;-------------------------------------------------
        ; Append the '$' terminator to the string.
        ;-------------------------------------------------
        MOV BYTE PTR [SI], '$'  ; Terminate the string for DOS function 09h.

ConvertDone:
        ;-------------------------------------------------
        ; Restore the registers that were saved.
        ;-------------------------------------------------
        POP SI           ; Restore SI.
        POP DX           ; Restore DX.
        POP CX           ; Restore CX.
        POP BX           ; Restore BX.
        RET              ; Return from the subroutine.
ConvertToString ENDP

        END main

```

q4
```
.model small
.stack 100h
.data
    ; DOS buffered input structure:
    ; Byte 0: Maximum number of characters allowed.
    ; Byte 1: Actual number of characters read.
    ; Bytes 2...: The characters typed by the user.
    inputBuffer  db 100, ?, 100 dup('$')
    
    ; Temporary buffer to hold a single word.
    ; We terminate the word with '$' for DOS function 09h.
    wordBuffer   db 100 dup('$')
    
    currRow      db 0         ; (Not used in this example but available)
    ;screenAttr   db 13h       ; (Blue background, cyan text attribute, not used by DOS 09h)
    
    prompt       db "enter a sentence:$"
    space        db ' ', '$'  ; A single space (for printing leading spaces)
    enter        db 0dh, 0ah, '$'  ; Newline string
.code
main proc
    ; Initialize data segment
    mov ax, @data
    mov ds, ax

    ; Clear the screen
    call clearscreen
    
    mov ah, 0bh
	mov bh, 00h
	mov bl, 1eh
	int 10h

    ; Display the prompt
    lea dx, prompt
    call display

    ; Read input using DOS buffered input (function 0Ah)
    mov ah, 0ah
    lea dx, inputBuffer
    int 21h
    call newline

    ; Process the input string (split into words and display each centered)
    call processing

    ; Exit to DOS
    mov ah, 4Ch
    int 21h
main endp

;-----------------------------------------------------------
; processing proc
;   This procedure processes the input string. It uses the
;   count of characters (from inputBuffer[1]) to loop through
;   the input. Words are delimited by spaces or a carriage return.
;   Each collected word is terminated with '$', then displayed
;   centered, followed by a newline.
;-----------------------------------------------------------
processing proc
    lea si, inputBuffer+2       ; SI points to the first character of input
    mov cl, [inputBuffer+1]     ; CL = number of characters read
    mov ch, 0                   ; Clear CH so that CX holds the full count
    mov di, 0                   ; DI will serve as a word length counter
process_loop:
    cmp cx, 0                   ; If no more characters, we're done
    je finish
    dec cx                      ; Decrement count (processing one character)
    lodsb                       ; Load next character from [SI] into AL (SI auto-increments)
    cmp al, 0dh                 ; Check for carriage return (enter key)
    je displayword
    cmp al, ' '                 ; Check for a space delimiter
    je displayword

    ; Not a delimiter: store character in wordBuffer.
    mov [wordBuffer+di], al
    inc di                      ; Increment word length counter
    jmp process_loop

displayword:
    cmp di, 0                   ; If no characters in word, skip display
    je skip_delimiter
    mov [wordBuffer+di], '$'    ; Terminate the word for display
    call center                 ; Display the word centered
    mov di, 0                   ; Reset word length for the next word
    call newline                ; Print a newline after the word
skip_delimiter:
    jmp process_loop            ; Continue processing remaining characters

finish:
    ; If a word is still pending (no delimiter at end), display it.
    cmp di, 0
    je done_process
    mov [wordBuffer+di], '$'
    call center
    call newline
done_process:
    ret
processing endp

;-----------------------------------------------------------
; clearscreen proc
;   Clears the screen by setting text mode (mode 03h).
;-----------------------------------------------------------
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

;-----------------------------------------------------------
; display proc
;   Displays a '$'-terminated string whose address is in DX.
;-----------------------------------------------------------
display proc
    mov ah, 09h
    int 21h
    ret
display endp

;-----------------------------------------------------------
; newline proc
;   Prints a newline.
;-----------------------------------------------------------
newline proc
    lea dx, enter
    call display
    ret
newline endp

;-----------------------------------------------------------
; center proc
;   Centers the string stored in wordBuffer on an 80-column screen.
;   It computes the left margin as (80 - word_length) / 2 and prints
;   that many spaces before printing the word.
;   Input: DI holds the word length.
;-----------------------------------------------------------
center proc
    push ax
    push bx
    push cx

    mov cx, di            ; CX = word length
    mov ax, 80            ; Total columns = 80
    sub ax, cx            ; AX = 80 - word length
    shr ax, 1             ; Left margin = (80 - word length) / 2
    mov bx, ax            ; BX = number of spaces to print

printspaces:
    cmp bx, 0
    je printword
    lea dx, space
    call display
    dec bx
    jmp printspaces

printword:
    lea dx, wordBuffer    ; Load address of the word
    call display
    pop cx
    pop bx
    pop ax
    ret
center endp

end main


```


q5
```
.MODEL SMALL
.STACK 100H
.DATA
numarray db 5, 3, 4, 2, 6, -1  ; Using -1 as an end marker
table db 40 dup(?)             ; Space for string output
newline1 db 0Dh, 0Ah, '$'       ; Newline string

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV SI, 0                 ; SI points to numarray

    CALL CLEARSCREEN

PASS:
    MOV AL, [numarray + SI]   ; Load number
    CMP AL, -1                ; Check for end marker (-1)
    JE TERMINATE              ; If -1, terminate loop

    MOV CX, 10                ; Loop counter (10 multiples)
    MOV DI, 0                 ; Reset table index
    MOV BL, AL                ; Store original value in BL

ITERATE:
    ; Calculate multiplier as (11 - CX) to get 1 to 10
    MOV AX, 11                ; Load 11 into AX
    SUB AX, CX                ; Subtract CX (which is 10,9,...,1)
    MUL BL                    ; Multiply BL by (11 - CX)
    CALL NUM_TO_STRING        ; Convert result in AX to ASCII and store in table

    MOV [table + DI], ' '     ; Add space
    INC DI
    LOOP ITERATE              ; Repeat for 10 multiples

    MOV [table + DI], '$'     ; Null-terminate table
    LEA DX, table
    CALL DISPLAY              ; Print multiplication table
    CALL NEWLINE              ; Print newline

    INC SI                    ; Move to next number
    JMP PASS                  ; Repeat

TERMINATE:
    CALL NEWLINE
    MOV AH, 4CH               ; Exit program
    INT 21H

MAIN ENDP

; Procedure to clear screen
CLEARSCREEN PROC
    MOV AH, 00H
    MOV AL, 03H
    INT 10H
    RET
CLEARSCREEN ENDP

; Procedure to display a string
DISPLAY PROC
    MOV AH, 09H
    INT 21H
    RET
DISPLAY ENDP

; Procedure to print a new line
NEWLINE PROC
    LEA DX, newline1
    CALL DISPLAY
    RET
NEWLINE ENDP

; Convert AX (binary number) to ASCII string in 'table'
NUM_TO_STRING PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    MOV CX, 0                 ; Digit counter
CONVERT_LOOP:
    MOV DX, 0                 ; Clear DX before division
    MOV BX, 10
    DIV BX                    ; AX / 10, remainder in DX
    ADD DL, '0'               ; Convert remainder to ASCII
    PUSH DX                   ; Store ASCII character on stack
    INC CX                    ; Increment digit counter
    TEST AX, AX               ; Check if AX == 0
    JNZ CONVERT_LOOP          ; If not, continue

STORE_DIGITS:
    POP DX                    ; Get ASCII digit
    MOV [table + DI], DL      ; Store in table
    INC DI
    LOOP STORE_DIGITS         ; Loop for all stored digits

    POP DX
    POP CX
    POP BX
    POP AX
    RET
NUM_TO_STRING ENDP

END MAIN
```

q6
```
.model small
.stack 100h
.data
    ; DOS buffered input structure:
    ; Byte 0: Maximum number of characters allowed.
    ; Byte 1: Actual number of characters read.
    ; Bytes 2...: The characters typed by the user.
    inputBuffer  db 100, ?, 100 dup('$')
    
    ; Temporary buffer to hold a single word.
    ; We terminate the word with '$' for DOS function 09h.
    wordBuffer   db 100 dup('$')
    
    currRow      db 0         ; (Not used in this example but available)
    ;screenAttr   db 13h       ; (Blue background, cyan text attribute, not used by DOS 09h)
    
    prompt       db "enter a sentence:$"
    space        db ' ', '$'  ; A single space (for printing leading spaces)
    enter        db 0dh, 0ah, '$'  ; Newline string
.code
main proc
    ; Initialize data segment
    mov ax, @data
    mov ds, ax

    ; Clear the screen
    call clearscreen
    
    	;mov ah, 0bh
	;mov bh, 00h
	;mov bl, 1eh
	;int 10h

    ; Display the prompt
    lea dx, prompt
    call display

    ; Read input using DOS buffered input (function 0Ah)
    mov ah, 0ah
    lea dx, inputBuffer
    int 21h
    call newline

    ; Process the input string (split into words and display each centered)
    call processing

    ; Exit to DOS
    mov ah, 4Ch
    int 21h
main endp

;-----------------------------------------------------------
; processing proc
;   This procedure processes the input string. It uses the
;   count of characters (from inputBuffer[1]) to loop through
;   the input. Words are delimited by spaces or a carriage return.
;   Each collected word is terminated with '$', then displayed
;   centered, followed by a newline.
;-----------------------------------------------------------
processing proc
    lea si, inputBuffer+2       ; SI points to the first character of input
    mov cl, [inputBuffer+1]     ; CL = number of characters read
    mov ch, 0                   ; Clear CH so that CX holds the full count
    mov di, 0                   ; DI will serve as a word length counter
process_loop:
    cmp cx, 0                   ; If no more characters, we're done
    je finish
    dec cx                      ; Decrement count (processing one character)
    lodsb                       ; Load next character from [SI] into AL (SI auto-increments)
    cmp al, 0dh                 ; Check for carriage return (enter key)
    je displayword
    cmp al, ' '                 ; Check for a space delimiter
    je displayword

    ; Not a delimiter: store character in wordBuffer.
    mov [wordBuffer+di], al
    inc di                      ; Increment word length counter
    jmp process_loop

displayword:
    cmp di, 0                   ; If no characters in word, skip display
    je skip_delimiter
    mov [wordBuffer+di], '$'    ; Terminate the word for display
    call center                 ; Display the word centered
    mov di, 0                   ; Reset word length for the next word
    call newline                ; Print a newline after the word
skip_delimiter:
    jmp process_loop            ; Continue processing remaining characters

finish:
    ; If a word is still pending (no delimiter at end), display it.
    cmp di, 0
    je done_process
    mov [wordBuffer+di], '$'
    call center
    call newline
done_process:
    ret
processing endp

;-----------------------------------------------------------
; clearscreen proc
;   Clears the screen by setting text mode (mode 03h).
;-----------------------------------------------------------
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

;-----------------------------------------------------------
; display proc
;   Displays a '$'-terminated string whose address is in DX.
;-----------------------------------------------------------
display proc
    mov ah, 09h
    int 21h
    ret
display endp

;-----------------------------------------------------------
; newline proc
;   Prints a newline.
;-----------------------------------------------------------
newline proc
    lea dx, enter
    call display
    ret
newline endp

;-----------------------------------------------------------
; center proc
;   Centers the string stored in wordBuffer on an 80-column screen.
;   It computes the left margin as (80 - word_length) / 2 and prints
;   that many spaces before printing the word.
;   Input: DI holds the word length.
;-----------------------------------------------------------
center proc
    push ax
    push bx
    push cx

    mov cx, di            ; CX = word length
    mov ax, 80            ; Total columns = 80
    sub ax, cx            ; AX = 80 - word length
    shr ax, 1             ; Left margin = (80 - word length) / 2
    mov bx, ax            ; BX = number of spaces to print

printspaces:
    cmp bx, 0
    je printword
    lea dx, space
    call display
    dec bx
    jmp printspaces

printword:
    lea dx, wordBuffer    ; Load address of the word
    call display
    pop cx
    pop bx
    pop ax
    ret
center endp

end main

```


q7

```
.model small
.stack 100h
.data
    msg1        db "Enter a string: $"
    msg2        db "Converted string: $"
    msg3        db "No. of uppercase: $"
    upper       db 0                  ; Initialize count to 0
    count_str   db 3 dup('$')         ; Holds ASCII representation of count
    string      db 50, ?, 50 dup('$') ; Input buffer
    enter       db 0dh, 0ah, '$'

.code
main proc
    mov ax, @data
    mov ds, ax
    
    call clearscreen
    
    ; Prompt for input
    lea dx, msg1
    call display
    
    ; Read string
    mov ah, 0ah
    lea dx, string
    int 21h
    
    ; Replace carriage return with '$'
    lea bx, string+1        ; Get input length
    mov byte ptr [string+2 + bx], '$'
    
    call newline
    
    ; Display converted string message
    lea dx, msg2
    call display
    call newline
    
    ; Process the string (convert vowels to uppercase)
    call processing
    
    ; Display converted string
    lea dx, string+2
    call display
    call newline
    
    ; Count uppercase letters
    call counting
    
    ; Convert count to ASCII string
    call convert_upper_to_ascii
    
    ; Display count message and result
    lea dx, msg3
    call display
    lea dx, count_str
    call display
    call newline
    
    ; Exit program
    mov ah, 4ch
    int 21h
main endp

; Converts lowercase vowels to uppercase
processing proc
    lea si, string+2        ; Start of string data
    mov cl, string+1        ; Length of string
    mov ch, 0
    jcxz terminate          ; Exit if empty string
next_char:
    mov al, [si]
    ; Check for lowercase vowels
    cmp al, 'a'
    je convert
    cmp al, 'e'
    je convert
    cmp al, 'i'
    je convert
    cmp al, 'o'
    je convert
    cmp al, 'u'
    je convert
    jmp continue
convert:
    sub al, 20h            ; Convert to uppercase
    mov [si], al
continue:
    inc si
    loop next_char
terminate:
    ret
processing endp

; Counts uppercase letters in the string
counting proc
    mov upper, 0           ; Reset count
    lea si, string+2       ; Start of string data
    mov cl, string+1       ; Length of string
    mov ch, 0
    jcxz done              ; Exit if empty string
count_loop:
    mov al, [si]
    cmp al, 'A'
    jb skip                ; Below 'A'
    cmp al, 'Z'
    ja skip                ; Above 'Z'
    inc upper              ; Increment count
skip:
    inc si
    loop count_loop
done:
    ret
counting endp

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

; Clears the screen
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

; Displays string pointed by DX
display proc
    mov ah, 09h
    int 21h
    ret
display endp

; Prints newline
newline proc
    lea dx, enter
    call display
    ret
newline endp

end main
```

q8
```
.model small
.stack 100h
.data
    msg1        db "Enter a string: $"
    ; Note: Changed the message to indicate the reversed string
    msg2        db "Reversed string without vowels: $"
    msg3        db "No. of vowels: $"
    vowel_count db 0                  ; Initialize vowel count to 0
    count_str   db 3 dup('$')         ; Holds ASCII representation of count
    string      db 50, ?, 50 dup('$') ; Input buffer
    enter       db 0dh, 0ah, '$'

.code
main proc
    mov ax, @data
    mov ds, ax
    
    call clearscreen
    
    ; Prompt for input
    lea dx, msg1
    call display
    
    ; Read string using DOS buffered input
    mov ah, 0ah
    lea dx, string
    int 21h
    
    ; Replace carriage return with '$'
    lea bx, string+1        ; BX = length of input
    mov byte ptr [string+2 + bx], '$'
    
    call newline
    
    ; Process the string (remove vowels and count them)
    call processing

    ; Reverse the processed string (stored at string+2)
    call reverse
    
    ; Display the reversed string message
    lea dx, msg2
    call display
    call newline
    
    ; Display the reversed string without vowels
    lea dx, string+2
    call display
    call newline
    
    ; Convert vowel count to an ASCII string
    call convert_vowel_count_to_ascii
    
    ; Display vowel count message and result
    lea dx, msg3
    call display
    lea dx, count_str
    call display
    call newline
    
    ; Exit program
    mov ah, 4Ch
    int 21h
main endp

;----------------------------------------------------------
; PROCESSING: Remove vowels from the input string and count them.
; The processed string (with vowels removed) is stored back into
; the same buffer (starting at string+2). It is terminated by '$'.
;----------------------------------------------------------
processing proc
    lea si, string+2        ; Start of input string data
    lea di, string+2        ; Destination for string without vowels
    mov cl, [string+1]      ; Number of characters entered
    mov ch, 0
    jcxz terminate          ; Exit if empty string
next_char:
    mov al, [si]
    ; Check for vowels (both lowercase and uppercase)
    cmp al, 'a'
    je is_vowel
    cmp al, 'e'
    je is_vowel
    cmp al, 'i'
    je is_vowel
    cmp al, 'o'
    je is_vowel
    cmp al, 'u'
    je is_vowel
    cmp al, 'A'
    je is_vowel
    cmp al, 'E'
    je is_vowel
    cmp al, 'I'
    je is_vowel
    cmp al, 'O'
    je is_vowel
    cmp al, 'U'
    je is_vowel
    ; Not a vowel – copy the character
    mov [di], al
    inc di
    jmp continue
is_vowel:
    inc vowel_count         ; Increment vowel count
continue:
    inc si
    loop next_char
terminate:
    mov byte ptr [di], '$'  ; Terminate the new string
    ret
processing endp

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

;----------------------------------------------------------
; CONVERT_VOWEL_COUNT_TO_ASCII: Convert the numeric vowel count
; (stored in vowel_count) to an ASCII string in count_str.
;----------------------------------------------------------
convert_vowel_count_to_ascii proc
    mov al, vowel_count
    aam                   ; Divide AL by 10; quotient in AH, remainder in AL
    add ax, '00'          ; Convert both digits to their ASCII codes
    ; Handle single-digit case
    cmp ah, '0'
    jne two_digits
    mov count_str[0], al
    mov count_str[1], '$'
    jmp exit_conv
two_digits:
    mov count_str[0], ah  ; Tens digit
    mov count_str[1], al  ; Ones digit
    mov count_str[2], '$'
exit_conv:
    ret
convert_vowel_count_to_ascii endp

;----------------------------------------------------------
; CLEARSCREEN: Clear the screen by setting the video mode.
;----------------------------------------------------------
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

;----------------------------------------------------------
; DISPLAY: Display a '$'-terminated string pointed to by DX.
;----------------------------------------------------------
display proc
    mov ah, 09h
    int 21h
    ret
display endp

;----------------------------------------------------------
; NEWLINE: Print a carriage return/line feed.
;----------------------------------------------------------
newline proc
    lea dx, enter
    call display
    ret
newline endp

end main

```

q9
```
.model small
.stack 100h
.data
    msg1        db "Enter a string: $"
    msg2        db "String: $"
    msg3        db "No. of vowels: $"
    vowel_count db 0                  ; Initialize vowel count to 0
    count_str   db 3 dup('$')         ; Holds ASCII representation of count
    string      db 50, ?, 50 dup('$') ; Input buffer
    enter       db 0dh, 0ah, '$'

.code
main proc
    mov ax, @data
    mov ds, ax
    
    call clearscreen
    
    ; Prompt for input
    lea dx, msg1
    call display
    
    ; Read string
    mov ah, 0ah
    lea dx, string
    int 21h
    
    ; Replace carriage return with '$'
    lea bx, string+1        ; Get input length
    mov byte ptr [string+2 + bx], '$'
    
    call newline
    
    ; Process the string (count vowels and remove them)
    call processing
    
    call clearscreen
    
    ; Display string without vowels message
    lea dx, msg2
    call display
    call newline
    
    ; Display string without vowels
    lea dx, string+2
    call display
    call newline
    
    ; Convert vowel count to ASCII string
    call convert_vowel_count_to_ascii
    
    ; Display vowel count message and result
    lea dx, msg3
    call display
    lea dx, count_str
    call display
    call newline
    
    ; Exit program
    mov ah, 4ch
    int 21h
main endp

; Counts vowels and removes them from the string
processing proc
    lea si, string+2        ; Start of string data
    ;lea di, string+2              ; Destination for string without vowels
    mov cl, string+1        ; Length of string
    mov ch, 0
    jcxz terminate          ; Exit if empty string
next_char:
    mov al, [si]
    ; Check for vowels (both lowercase and uppercase)
    cmp al, 'a'
    je is_vowel
    cmp al, 'e'
    je is_vowel
    cmp al, 'i'
    je is_vowel
    cmp al, 'o'
    je is_vowel
    cmp al, 'u'
    je is_vowel
    cmp al, 'A'
    je is_vowel
    cmp al, 'E'
    je is_vowel
    cmp al, 'I'
    je is_vowel
    cmp al, 'O'
    je is_vowel
    cmp al, 'U'
    je is_vowel
    
    jmp continue
is_vowel:
    inc vowel_count         ; Increment vowel count
continue:
    inc si
    loop next_char
terminate:
   
    ret
processing endp

; Converts numeric vowel count to ASCII string
convert_vowel_count_to_ascii proc
    mov al, vowel_count
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
convert_vowel_count_to_ascii endp

; Clears the screen
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

; Displays string pointed by DX
display proc
    mov ah, 09h
    int 21h
    ret
display endp

; Prints newline
newline proc
    lea dx, enter
    call display
    ret
newline endp

end main
```
q10
```
.model small
.stack 100h
.data
    numbers db 9, 2, 5, 1, 7, 4, 8, 3, 6, 0  ; Array of 10 numbers to be sorted
    count   dw 10                             ; Number of elements in the array
    msg1    db "Sorted numbers: $"
    space   db ' $'                           ; Space for separating numbers
    newline db 0dh, 0ah, '$'                  ; Newline string

.code
main proc
    ; Initialize data segment
    mov ax, @data
    mov ds, ax

    ; Sort the numbers using Bubble Sort
    call bubble_sort

    ; Display the sorted numbers
    call display_numbers

    ; Exit to DOS
    mov ah, 4Ch
    int 21h
main endp

;-----------------------------------------------------------
; bubble_sort proc
;   Sorts the array of numbers in ascending order using
;   the Bubble Sort algorithm.
;-----------------------------------------------------------
bubble_sort proc
    mov cx, count                ; CX = number of elements
    dec cx                       ; CX = number of passes (n-1)
outer_loop:
    mov si, 0                    ; SI = index for array traversal
    mov dx, 0                    ; DX = flag to check if any swap occurred
inner_loop:
    mov al, [numbers+si]          ; AL = current element
    mov bl, [numbers+si+1]        ; BL = next element
    cmp al, bl                   ; Compare current and next element
    jbe no_swap                  ; If AL <= BL, no swap needed
    ; Swap AL and BL
    mov [numbers+si], bl
    mov [numbers+si+1], al
    mov dx, 1                    ; Set swap flag
no_swap:
    inc si                       ; Move to the next element
    cmp si, cx                   ; Check if we're at the last element
    jl inner_loop                ; Repeat for the remaining elements
    cmp dx, 0                    ; Check if any swap occurred
    je sorted                    ; If no swaps, array is sorted
    dec cx                       ; Decrement pass counter
    jnz outer_loop               ; Repeat for the next pass
sorted:
    ret
bubble_sort endp

;-----------------------------------------------------------
; display_numbers proc
;   Displays the sorted numbers on the screen.
;-----------------------------------------------------------
display_numbers proc
    ; Display the message
    lea dx, msg1
    mov ah, 09h
    int 21h

    ; Display the sorted numbers
    mov cx, count                ; CX = number of elements
    mov si, 0                    ; SI = index for array traversal
display_loop:
    mov al, [numbers+si]          ; AL = current number
    call display_number           ; Display the number
    lea dx, space                ; Display a space
    mov ah, 09h
    int 21h
    inc si                       ; Move to the next number
    loop display_loop            ; Repeat for all numbers

    ; Display a newline
    lea dx, newline
    mov ah, 09h
    int 21h
    ret
display_numbers endp

;-----------------------------------------------------------
; display_number proc
;   Displays a single-digit or two-digit number in AL.
;-----------------------------------------------------------
display_number proc
    push ax
    push bx
    push cx
    push dx

    ; Convert AL to ASCII
    mov ah, 0                    ; Clear AH
    mov bl, 10                   ; BL = 10 (divisor)
    div bl                       ; AL = quotient (tens digit), AH = remainder (units digit)

    ; Display tens digit (if any)
    cmp al, 0                    ; If quotient (tens digit) is zero, skip
    je skip_tens
    add al, '0'                  ; Convert to ASCII
    mov dl, al                   ; DL = tens digit
    mov ah, 02h                  ; DOS display character function
    int 21h
skip_tens:
    add ah, '0'                  ; Convert units digit (remainder) to ASCII
    mov dl, ah                   ; DL = units digit
    mov ah, 02h                  ; DOS display character function
    int 21h

    pop dx
    pop cx
    pop bx
    pop ax
    ret
display_number endp

end main


```

q11
```
.model small
.stack 100h
.data
    msg1        db "Enter a string: $"
    msg2        db "Alphabetic characters only: $"
    string      db 50, ?, 50 dup('$') ; Input buffer
    enter       db 0dh, 0ah, '$'

.code
main proc
    mov ax, @data
    mov ds, ax
    
    call clearscreen
    
    ; Prompt for input
    lea dx, msg1
    call display
    
    ; Read string
    mov ah, 0ah
    lea dx, string
    int 21h
    
    ; Replace carriage return with '$'
    lea bx, string+1        ; Get input length
    mov byte ptr [string+2 + bx], '$'
    
    call newline
    
    ; Process the string (remove non-alphabetic characters)
    call filter_alphabetic
    
    ; Clear the screen
    call clearscreen
    
    ; Display filtered string message
    lea dx, msg2
    call display
    call newline
    
    ; Display filtered string
    lea dx, string+2
    call display
    call newline
    
    ; Exit program
    mov ah, 4ch
    int 21h
main endp

; Removes non-alphabetic characters from the string
filter_alphabetic proc
    lea si, string+2        ; Start of string data
    lea di, string+2             ; Destination for filtered string
    mov cl, string+1        ; Length of string
    mov ch, 0
    jcxz terminate          ; Exit if empty string
next_char:
    mov al, [si]
    ; Check if the character is alphabetic (A-Z or a-z)
    cmp al, 'A'
    jb skip                 ; Below 'A'
    cmp al, 'Z'
    jbe copy_char           ; Between 'A' and 'Z'
    cmp al, 'a'
    jb skip                 ; Below 'a'
    cmp al, 'z'
    ja skip                 ; Above 'z'
copy_char:
    mov [di], al            ; Copy alphabetic character
    inc di
skip:
    inc si
    loop next_char
terminate:
    mov byte ptr [di], '$'  ; Null-terminate the filtered string
    ret
filter_alphabetic endp

; Clears the screen
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

; Displays string pointed by DX
display proc
    mov ah, 09h
    int 21h
    ret
display endp

; Prints newline
newline proc
    lea dx, enter
    call display
    ret
newline endp

end main

```

q12
```
.model small
.stack 100h
.data
    ; DOS buffered input structure:
    ; Byte 0: Maximum number of characters allowed.
    ; Byte 1: Actual number of characters read.
    ; Bytes 2...: The characters typed by the user.
    inputBuffer  db 100, ?, 100 dup('$')
    
    ; Temporary buffer to hold a single word.
    ; We terminate the word with '$' for DOS function 09h.
    wordBuffer   db 100 dup('$')
    
    prompt       db "Enter a sentence: $"
    space        db ' ', '$'  ; A single space (for printing leading spaces)
    enter        db 0dh, 0ah, '$'  ; Newline string
    wordCountMsg db "Number of words: $"
    wordCount    dw 0         ; Counter for the number of words
.code
main proc
    ; Initialize data segment
    mov ax, @data
    mov ds, ax

    ; Clear the screen
    call clearscreen

    ; Display the prompt
    lea dx, prompt
    call display

    ; Read input using DOS buffered input (function 0Ah)
    mov ah, 0ah
    lea dx, inputBuffer
    int 21h
    call newline

    ; Process the input string (split into words and display each)
    call processing

    ; Display the word count
    lea dx, wordCountMsg
    call display
    mov ax, wordCount
    call display_number
    call newline

    ; Exit to DOS
    mov ah, 4Ch
    int 21h
main endp

;-----------------------------------------------------------
; processing proc
;   This procedure processes the input string. It uses the
;   count of characters (from inputBuffer[1]) to loop through
;   the input. Words are delimited by spaces or a carriage return.
;   Each collected word is terminated with '$', then displayed,
;   followed by a newline.
;-----------------------------------------------------------
processing proc
    lea si, inputBuffer+2       ; SI points to the first character of input
    mov cl, [inputBuffer+1]     ; CL = number of characters read
    mov ch, 0                   ; Clear CH so that CX holds the full count
    mov di, 0                   ; DI will serve as a word length counter
process_loop:
    cmp cx, 0                   ; If no more characters, we're done
    je finish
    dec cx                      ; Decrement count (processing one character)
    lodsb                       ; Load next character from [SI] into AL (SI auto-increments)
    cmp al, 0dh                 ; Check for carriage return (enter key)
    je displayword
    cmp al, ' '                 ; Check for a space delimiter
    je displayword

    ; Not a delimiter: store character in wordBuffer.
    mov [wordBuffer+di], al
    inc di                      ; Increment word length counter
    jmp process_loop

displayword:
    cmp di, 0                   ; If no characters in word, skip display
    je skip_delimiter
    mov [wordBuffer+di], '$'    ; Terminate the word for display
    call display_word           ; Display the word
    inc wordCount               ; Increment word count
    mov di, 0                   ; Reset word length for the next word
    call newline                ; Print a newline after the word
skip_delimiter:
    jmp process_loop            ; Continue processing remaining characters

finish:
    ; If a word is still pending (no delimiter at end), display it.
    cmp di, 0
    je done_process
    mov [wordBuffer+di], '$'
    call display_word
    inc wordCount               ; Increment word count
    call newline
done_process:
    ret
processing endp

;-----------------------------------------------------------
; clearscreen proc
;   Clears the screen by setting text mode (mode 03h).
;-----------------------------------------------------------
clearscreen proc
    mov ah, 00h
    mov al, 03h
    int 10h
    ret
clearscreen endp

;-----------------------------------------------------------
; display proc
;   Displays a '$'-terminated string whose address is in DX.
;-----------------------------------------------------------
display proc
    mov ah, 09h
    int 21h
    ret
display endp

;-----------------------------------------------------------
; newline proc
;   Prints a newline.
;-----------------------------------------------------------
newline proc
    lea dx, enter
    call display
    ret
newline endp

;-----------------------------------------------------------
; display_word proc
;   Displays the word stored in wordBuffer.
;-----------------------------------------------------------
display_word proc
    lea dx, wordBuffer    ; Load address of the word
    call display
    ret
display_word endp

;-----------------------------------------------------------
; display_number proc
;   Displays the number in AX as a decimal.
;-----------------------------------------------------------
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