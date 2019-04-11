HW02
===
This is the hw02 sample. Please follow the steps below.

# Build the Sample Program

1. Fork this repo to your own github account.

2. Clone the repo that you just forked.

3. Under the hw02 dir, use:

	* `make` to build.

	* `make clean` to clean the ouput files.

4. Extract `gnu-mcu-eclipse-qemu.zip` into hw02 dir. Under the path of hw02, start emulation with `make qemu`.

	See [Lecture 02 ─ Emulation with QEMU] for more details.

5. The sample is designed to help you to distinguish the main difference between the `b` and the `bl` instructions.  

	See [ESEmbedded_HW02_Example] for knowing how to do the observation and how to use markdown for taking notes.

# Build Your Own Program

1. Edit main.s.

2. Make and run like the steps above.

# HW02 Requirements

1. Please modify main.s to observe the `push` and the `pop` instructions:  

	Does the order of the registers in the `push` and the `pop` instructions affect the excution results?  

	For example, will `push {r0, r1, r2}` and `push {r2, r0, r1}` act in the same way?  

	Which register will be pushed into the stack first?

2. You have to state how you designed the observation (code), and how you performed it.  

	Just like how [ESEmbedded_HW02_Example] did.

3. If there are any official data that define the rules, you can also use them as references.

4. Push your repo to your github. (Use .gitignore to exclude the output files like object files or executable files and the qemu bin folder)

[Lecture 02 ─ Emulation with QEMU]: http://www.nc.es.ncku.edu.tw/course/embedded/02/#Emulation-with-QEMU
[ESEmbedded_HW02_Example]: https://github.com/vwxyzjimmy/ESEmbedded_HW02_Example

--------------------

- [] **If you volunteer to give the presentation next week, check this.**

--------------------

{push}和{pop}中暫存器的順序是否會影響結果?

//這兩指令執行結果會一樣嗎?
push {r0, r1, r2} 
push {r2, r0, r1}

那哪個暫存器會先被推入stack?

![](http://md.nc.es.ncku.edu.tw/uploads/upload_f68f2434240ec4e4d8fa23fa5fb9cd91.png)

附圖是arm的stack堆疊方向原理，了解其原理後接著我們思考兩個指令

push{r1, r2, r3, r4}

這個指令是將r1, r2, r3, r4，還是r4, r3, r2, r1依照順序放入stack?

pop{r5, r6, r7, r8}

這個指令是將剛才存入stack的四個值放入r5,r6,r7,r8暫存器，
第一份從stack拿出的值會放到r8呢，還是r5?

在push與pop指令中，更改暫存器的順序影響?
我們將之前的push與pop指令中的暫存器順序倒至，會影響push與pop資料的順序嗎??

push{r4, r3, r2, r1}

pop{r8, r7, r6, r5}


為了觀察暫存器的順序是否會影響執行結果，設計兩組指令暫存器的順序剛好相反，利用**make qemu**模擬器，使用**si**指令可以觀察其資料存放的順序與結果。

結果

.syntax unified

.word 0x20000100
.word _start

.global _start
.type _start, %function
_start:
	
	//
	//mov # to reg
	//
	movs	r1,	#1
	movs	r2,	#2
	movs	r3,	#3
    movs    r4, #4
	
	//
	//push
	//
	push	{r1, r2, r3, r4}

    //
	//pop
	//
	pop     {r5, r6, r7, r8}

	//
	//clean the register data
	//
	movs    r5, #0
	movs    r6, #0
	movs    r7, #0
	movs    r8, #0

	//
	//push
	//
	push    {r4, r3, r2, r1}

	//
	//pop
	//
	pop     {r8, r7, r6, r5}

	//
	//b bl
	//
	bl	label01

sleep:
	b	sleep

label01:
	nop
	nop
	bx	lr


先將1,2,3,4放入r1,r2,r3,r4
接著，我們將r1,r2,r3,r4的值push到stack pointer(sp)
得知我們的sp向下堆疊到0x200000f0這個位置，我們利用x 0x200000f0來觀察我們最後放入的資料是哪一個暫存器的值???

0X20000100               //stack起始點 
0x200000fc   0x00000004  //為r4的值
0x200000f8   0x00000003  //為r3的值
0x200000f4   0x00000002  //為r2的值
0x200000f0   0x00000001  //為r1的值

由模擬器我們得知，PUSH{r1, r2, r3, r4}，會依序r4, r3, r2, r1的值往下堆疊，接著我們觀察POP{r5, r6, r7, r8}。
還記得push和pop的架構是**FILO**(First In Last Out)，因此當pop{r5, r6, r7, r8}指令執行時，會先去找sp也就是剛才的0x200000f0而其值為4，並將其放入R8，接著0x200000f4其值為3放入r7，以此類推，最後pop完，sp為0x20000100

0X20000100               //stack起始點 
0x200000fc   0x00000004  //pop到r8
0x200000f8   0x00000003  //pop到r7
0x200000f4   0x00000002  //pop到r6
0x200000f0   0x00000001  //pop到r5

了解push與pop後，接著是驗證

push{r1, r2, r3, r4} 與 push{r4, r3, r2, r1}
pop{r5, r6, r7, r8}  與 pop{r8, r7, r6, r5}

的差別，清空暫存器並執行si後發現，
系統將push{r4,r3,r2,r1}, pop{r8,r7,r6,r5}默認為push{r1,r2,r3 r4}, pop{r5,r6,r7,r8}!!



push

在registers中寫到，如果一次push多個registers，數字最小的暫存器會得到最低的記憶體位置，而數字最高的暫存器會得到最高的記憶體位置。
這也是為什麼，push{r1,r2,r3,r4} 會先push R4到stack的原因。
 
pop

在registers中寫到，如果一次pop多個registers，數字最小的暫存器會從最低的記憶體位置抓資料，而數字最高的暫存器會得到最高的記憶體位置的資料。這也是為什麼，pop{r5,r6,r7,r8} 會先讓r5去拿資料的原因。
    
由push與pop的架構我們得知，我們交換暫存器的順序是沒有用的
