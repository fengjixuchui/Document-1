## ʲô��IDT

IDT (Interrupt Descriptor Table) �ж�������

IDT���д洢��ISR(Interrupt Service Routines) �жϷ��������Щ���̻����жϲ���ʱ�����á�

IDT��һ����256����ڵ����Ա�ÿ��IDT������Ǹ�8�ֽڵ�����������������IDT��Ĵ�СΪ`256 * 8 = 2048 bytes`��**ÿ���ж�����������һ���жϴ�����̡���ν���ж��������ǰ�ÿ���жϻ����쳣��һ��0-255������ʶ��Intel���������Ϊ������**

����IDT������ϵͳʹ��IDTR�Ĵ�������¼IDT��λ�úʹ�С��

`IDTR`�Ĵ�����`48`λ�Ĵ��������ڱ���IDT����Ϣ������`��16λ`����IDT�Ĵ�С������Ϊ`0x7ff`��`��32λ`����IDT�Ļ���ַ������Ϊ`0x80b95400`��

**����ʹ��ָ��`sidt`����IDTR�Ĵ����е���Ϣ���Ӷ��ҵ�IDT���ڴ��е�λ�á�**

```C++
   typedef struct
   {
   WORD IDTLimit;
   WORD LowIDTbase;   //IDT�ĵͰ��ַ
   WORD HiIDTbase;     //IDT�ĸ߰��ַ
   } IDTINFO;
```

```C++
	IDTINFO idt_info; 
	  __asm{

         sidt idt_info;          //��ȡIDTINFO
 
      }
```


���ȿ�һ��`IDTR`��`IDTL`�Ĵ�����ֵ

```cmd
kd> r idtr
idtr=80b95400
kd> r idtr, idtl
idtr=80b95400 idtl=000007ff
```

## IDT HOOK

IDT,�жϾ���ͣ�����ڵĻ��ȥ����µ�����һ���жϿ���Դ�������Ӳ�����������ҳ���󣬵���IDT�е�0x0E�����û�����ϵͳ����SSDT��ʱ������IDT�е�0x2E����ϵͳ����ĵ����Ǿ����ģ�����жϾ��ܴ������������ھ���취������ϵͳ���ҵ�IDT��Ȼ��ȷ��0x2E��IDT�еĵ�ַ����������ǵĺ�����ַȥȡ����������һ�����û��Ľ���һ����SSDT�����ǵ�HOOK��������������

```C++
#pragma pack(1)
typedef struct
{
   WORD LowOffset;            //��ڵĵͰ��ַ
   WORD selector;
   BYTE unused_lo;
   unsigned char unused_hi:5;      // stored TYPE ?
   unsigned char DPL:2;
   unsigned char P:1;          // vector is present
   WORD HiOffset;           //��ڵ�ַ�ĵͰ��ַ
} IDTENTRY;
#pragma pack()
```

```C++
 int HookInterrupts()
   {
 
      IDTINFO idt_info;           //SIDT�����صĽṹ
 
      IDTENTRY* idt_entries;     //IDT���������
 
      IDTENTRY* int2e_entry;     //����Ŀ������
 
      __asm{
 
         sidt idt_info;          //��ȡIDTINFO
 
      }
     //��ȡ���е����
      idt_entries =
 
     (IDTENTRY*)MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);
 
   //������ʵ��2e��ַ
      KiRealSystemServiceISR_Ptr =
                                 MAKELONG(idt_entries[NT_SYSTEM_SERVICE_INT].LowOffset,
 
            idt_entries[NT_SYSTEM_SERVICE_INT].HiOffset);
 
   //��ȡ0x2E����ڵ�ַ
      int2e_entry = &(idt_entries[NT_SYSTEM_SERVICE_INT]);
 
      __asm{
 
        cli;                        // �����жϣ���ֹ������
 
        lea eax,MyKiSystemService; // �������hook�����ĵ�ַ��������eax
 
        mov ebx, int2e_entry;       // 0x2E��IDT�еĵ�ַ
 
        mov [ebx],ax;               // ������hook�����ĵ͵�ַд��͵�ַ
 
      shr eax,16                  //eax����16���õ��ߵ�ַ
 
        mov [ebx+6],ax;            // д��ߵ�ַ
 
      sti;                       //���ж�
 
      }
 
      return 0;
 
   }
```

## �����ж�

8086PC�����У����̵����뽫������9���жϣ�BIOS�ṩ��int 9���ж����̡�CPU��9���жϷ���֮�󣬻�ȥִ��`int 9`�ж����̣�Ȼ���`60h�˿�`���ж�ȡ��ɨ���룬���ҽ���ת��Ϊ��Ӧ��ASCII���״̬��Ϣ���洢���ڴ��ָ���Ŀռ�(���̻�������״̬�ֽ�)���С�

> һ��ļ������룬��CPUִ����int 9 �ж�����֮�󶼷ŵ��˼��̻��������У����̻�������`16���ֵ�Ԫ`�����Դ洢15������ɨ����Ͷ�Ӧ��ASCII�룬����֮����ֻ�ܷ�15������Ϊ���̻��������û��ζ��нṹ������ڴ�������Ȼ�������ı�����Ϊ16���֣��������жϡ����������Ŀ��ǣ������ֻ�ܱ���15��������Ϣ��

BIOS�ṩ��`int 16h`�ж����̹�����Ա���á���������а�����һ������Ҫ�Ĺ����ǴӼ��̻������ж�ȡһ���������롣

```asm
mov ah,0
int 16h
```

�����ah(ɨ����)��al(ascii��)
