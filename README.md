# Standard-Linux-Bootloader
This is a Linux Bootloader, edit as you see fit to use with your System (Pointers: Line 44 Message)

Copy and paste from line 6 to your text editor of choice call it "bootloader.bin" to make life simple :)

[BITS 16]            ; Tell assembler we're using 16-bit mode
[ORG 0x7C00]        ; Bootloader loads at 0x7C00

start:
    cli             ; Disable interrupts
    mov ax, 0x07C0 ; Set up segment registers
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0xFFFF ; Stack pointer
    sti             ; Enable interrupts

    ; Print message
    mov si, boot_msg
    call print_string

    ; Set up protected mode
    cli
    lgdt [gdt_descriptor]
    mov eax, cr0
    or eax, 1
    mov cr0, eax
    jmp CODE_SEG:init_protected_mode


; Function to print a string
global print_string
print_string:
    lodsb       ; Load next byte from SI into AL
    or al, al   ; Check if it's null terminator
    jz done     ; If zero, return
    mov ah, 0x0E ; BIOS teletype function
    int 0x10    ; Call BIOS interrupt
    jmp print_string

done:
    ret

boot_msg db 'Booting Linux...', 0

; GDT (Global Descriptor Table)
gdt_start:
    dq 0x0000000000000000 ; Null descriptor

gdt_code:
    dq 0x00CF9A000000FFFF ; Code segment

gdt_data:
    dq 0x00CF92000000FFFF ; Data segment

gdt_end:

gdt_descriptor:
    dw gdt_end - gdt_start - 1
    dd gdt_start

[BITS 32]
init_protected_mode:
    mov ax, DATA_SEG
    mov ds, ax
    mov es, ax
    mov fs, ax
    mov gs, ax
    mov ss, ax
    mov esp, 0x90000

    ; Jump to kernel (replace with actual Linux kernel loading code)
    jmp $   ; Infinite loop (halt system)

CODE_SEG equ gdt_code - gdt_start
DATA_SEG equ gdt_data - gdt_start

; Bootloader padding (make it 512 bytes)
times 510-($-$$) db 0

dw 0xAA55  ; Boot signature
