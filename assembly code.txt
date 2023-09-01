.data
A:      .word 0
B:      .word 0
C:      .word 0
Result: .word 0
Prompt: .asciiz "Enter value: "
Menu:   .asciiz "Select an equation (1-6):\n1. (A AND B) OR C\n2. (A OR B) AND (NOT C)\n3. (A XOR B) AND C\n4. (A + B) - C\n5. (A * B) + C\n6. (A / B) + (A % B) * C\n"
InvalidChoiceMsg: .asciiz "Invalid choice. Please enter a number again.\n"
ZeroDivisionError: .asciiz "Error: Division by zero\n"
ResultText: .asciiz "Result: "
ContinueMsg: .asciiz "\n\nDo you want to continue? (1: Yes, 2: No): "

.text
.globl main

main:
    li $v0, 4
    la $a0, Menu
    syscall

input_loop:
    li $v0, 4
    la $a0, Prompt
    syscall

    li $v0, 5
    syscall
    move $t0, $v0

    li $t1, 1
    li $t2, 6
    blt $t0, $t1, invalid_choice
    bgt $t0, $t2, invalid_choice

    beq $t0, 1, equation
    beq $t0, 2, equation
    beq $t0, 3, equation
    beq $t0, 4, equation
    beq $t0, 5, equation
    beq $t0, 6, equation
j continue

invalid_choice:
    li $v0, 4
    la $a0, InvalidChoiceMsg
    syscall
    j input_loop

equation:
    li $v0, 4
    la $a0, Prompt
    syscall
    li $v0, 5
    syscall
    sw $v0, A

    li $v0, 4
    la $a0, Prompt
    syscall
    li $v0, 5
    syscall
    sw $v0, B

    li $v0, 4
    la $a0, Prompt
    syscall
    li $v0, 5
    syscall
    sw $v0, C

    lw $t1, A
    lw $t2, B
    lw $t3, C

    beq $t0, 1, eq1
    beq $t0, 2, eq2
    beq $t0, 3, eq3
    beq $t0, 4, eq4
    beq $t0, 5, eq5
    beq $t0, 6, eq6

eq1:
    and $t4, $t1, $t2
    or $t4, $t4, $t3
    j done

eq2:
    or $t4, $t1, $t2
    not $t3, $t3
    and $t4, $t4, $t3
    j done

eq3:
    xor $t4, $t1, $t2
    and $t4, $t4, $t3
    j done

eq4:
    add $t4, $t1, $t2
    sub $t4, $t4, $t3
    j done

eq5:
    mul $t4, $t1, $t2
    add $t4, $t4, $t3
    j done

eq6:
    beqz $t2, divide_zero
    div $t1, $t2
    mflo $t4
    mfhi $t5
    mul $t5, $t5, $t3
    add $t4, $t4, $t5
    j done

divide_zero:
    li $v0, 4
    la $a0, ZeroDivisionError
    syscall
    j continue

done:
    li $v0, 4
    la $a0, ResultText
    syscall
    move $a0, $t4
    li $v0, 1
    syscall
j continue

continue: 
    li $v0, 4
    la $a0, ContinueMsg
    syscall
    li $v0, 5
    syscall
    move $t0, $v0
    beq $t0, 1, show_menu  # Show the main menu if user chooses to continue
    beq $t0, 2, exit_program  # Exit the program
    j continue  # Repeat the "done" loop if an invalid choice is made

show_menu:  
    li $v0, 4
    la $a0, Menu
    syscall    
    li $v0, 5
    move $t0, $v0
    j input_loop  # Jump to the input loop to get the user's choice for equation

exit_program:
    li $v0, 10
    syscall