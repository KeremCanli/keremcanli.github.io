---
layout: post
title: Girilen mXn Boyutlu Matrisin Transpozunu Alan Proc Uygulaması
---
<ul>
	<li>Modülü oluşturmak için gerekli c ve Makefile dosyası kodları aşağıdaki gibidir.</li>
	<h3>MatrixTransposer.c</h3>
	<pre>#include <linux/kernel.h> 
#include <linux/module.h>
#include <linux/proc_fs.h>
#include <linux/uaccess.h>
#include <linux/moduleparam.h>
#define ProcName "MatrixTransposer"
#define MaximumSize 2048
MODULE_LICENSE ("GPL");
MODULE_AUTHOR ("Module5.ko");
static int m; // Parametre olarak girilen integer ı tutar.
static int n; // Parametre olarak girilen integer ı tutar.
static unsigned long Size=0;
static char ProcBuffer[MaximumSize];
static struct proc_dir_entry *ProcDirEntry; // Proc ile ilgili bilgileri tutar.
module_param (m, int, 0000); // Parametre almak için kullanılan, değişken adı, değişken
türü, verilecek izinleri parametre olarak alan fonksiyon.
module_param (n, int, 0000); // Parametre almak için kullanılan, değişken adı, değişken
türü, verilecek izinleri parametre olarak alan fonksiyon.
MODULE_PARM_DESC (m, "Number Of Matrix Rows");
MODULE_PARM_DESC (n, "Number Of Matrix Columns");
static int Open (struct inode *Inode, struct file *File) // Proc u açan fonksiyon.
    {
        printk (KERN_INFO "Proc opened.\n");
 	return 0;
    }
static ssize_t Write (struct file *File, const char *Buffer, size_t Length, loff_t *Offset) // Proc a
yazan fonksiyon.
    {
        int i, j, k=0, l=0, Index=0;
	char Header1[6]="Matrix";
	char Header2[16]="Transpose Matrix";
	int CharacterNumbersOfInputs[m*n];
	char Inputs[m*n][100]; // En fazla 100 karakterli olabilen inputları tutar.		
 	if (Length>MaximumSize)
	    {
  	        Size=MaximumSize;
 	    }
 	else
	    {
  	        Size=Length;
 	    }
	for (i=0;i<=MaximumSize-1;i++)
            {
	        ProcBuffer[i]='\0'; // ProcBuffer dizisinin elemanlarını sıfırlar.
	    }
 	copy_from_user (ProcBuffer, Buffer, Size); // Veriyi Buffer dan ProcBuffer a kopyalar.
	for (i=0;i<=Size-1;i++) // ProcBuffer dizisini virgüle göre parçalayarak son input hariç
inputları Inputs dizisine atar ve son input hariç inputların karakter sayılarını
CharacterNumbersOfInputs dizisine atar.
	    {
	        if (ProcBuffer[i]==',')
		    {
		        for (j=k;j<=i-1;j++)
			    {
			        Inputs[l][Index]=ProcBuffer[j];
				Index++;
			    }
			CharacterNumbersOfInputs[l]=i-k;
			k=i+1;
			l++;
			Index=0;
		    }
	    }
	for (i=k;i<=Size-2;i++) // Son inputu Inputs dizisine atar.
	    {
	        Inputs[l][Index]=ProcBuffer[i];
		Index++;
	    }
	CharacterNumbersOfInputs[l]=Size-1-k; // Son inputun karakter sayısını
CharacterNumbersOfInputs dizisine atar.
	Index=0;
	Size=Size*2+24; // ProcBuffer dizisine matris ve transpoz matris atılacak olduğundan
dolayı Size ı 2 ile çarpar ve ProcBuffer dizisine "Matrix" ve "Transpose Matrix" başlıkları
atılacak olduğundan dolayı 24 ile toplar.
	for (i=0;i<=5;i++) // ProcBuffer dizisine "Matrix" string ini atar.
	    {
	        ProcBuffer[i]=Header1[i];
	    }
	ProcBuffer[6]='\n'; // Yeni satıra geçmek için ProcBuffer dizisine '\n' karakterini atar.
	Index=6;
	for (i=0;i<=(m*n)-1;i++)
	    {
	        for (j=0;j<=CharacterNumbersOfInputs[i]-1;j++) // Matrisin elemanlarını yani
inputları ProcBuffer dizisine atar.
		    {
		        Index++;
			ProcBuffer[Index]=Inputs[i][j];
		    }
		Index++;
		ProcBuffer[Index]='\t'; // Bir matris elemanını yani bir inputu ProcBuffer dizisine
attıktan sonra bir tab boşluk bırakmak için bu elemanın peşine '\t' karakterini atar.
		if ((i+1)%n==0)
		    {
		        ProcBuffer[Index]='\n'; // Bir matris elemanını yani bir inputu ProcBuffer
dizisine attıktan sonra bu eleman satır sonunda ise yeni satıra geçmek için peşine '\n' karakterini
atar.
		    }
	    }
	for (i=0;i<=15;i++) // ProcBuffer dizisine "Transpose Matrix" string ini atar.
	    {
	        Index++;
		ProcBuffer[Index]=Header2[i];
	    }
	Index++;
	ProcBuffer[Index]='\n'; // Yeni satıra geçmek için ProcBuffer dizisine '\n' karakterini atar.
	for (i=0;i<=n-1;i++)
	    {
	        for (j=i;j<=(m*n)-1;j=j+n)
		    {
		        for (k=0;k<=CharacterNumbersOfInputs[j]-1;k++)
			    {
			        Index++;
				ProcBuffer[Index]=Inputs[j][k]; // Bu üç döngü ile Inputs dizisinde
bulunan inputların transpozunu alır ve ProcBuffer dizisine atar.
			    }
			Index++;
			ProcBuffer[Index]='\t'; // Transpoz matrisin bir elemanını ProcBuffer dizisine
attıktan sonra bir tab boşluk bırakmak için bu elemanın peşine '\t' karakterini atar.
		    }
		ProcBuffer[Index]='\n'; // Transpoz matrisin bir satırını ProcBuffer dizisine attıktan
sonra yeni satıra geçmek için satır sonuna '\n' karakterini atar.
	    }
	printk (KERN_INFO "%s", ProcBuffer);
	printk (KERN_INFO "Write function wrote %lu bytes.\n", Size);
        return Size;
    }
static ssize_t Read (struct file *File, char *Buffer, size_t Length, loff_t *Offset) // Proc u
okuyan fonksiyon.
    {
        static int Finished=0;
 	if (Finished)
	    {
  	        printk (KERN_INFO "Read function finished reading.\n");
  		Finished=0;
  		return 0; // Dosyanın sonuna daha fazla bilgi yazmayacağını belirtmek için
return 0 yazılır.
 	    }
 	Finished=1;
 	copy_to_user (Buffer, ProcBuffer, Size); // Veriyi ProcBuffer dan Buffer a kopyalar.
	printk (KERN_INFO "%s", Buffer);
 	printk (KERN_INFO "Read function read %lu bytes.\n", Size);
 	return Size;
    }
static int Release (struct inode *Inode, struct file *File) // Proc u kapatan fonksiyon.
    {
        printk (KERN_INFO "Proc closed.\n");
 	return 0;
    }
static struct file_operations FileOperations={.open=Open,
					      .write=Write,
					      .read=Read,
					      .release=Release};
static int __init InitMatrixTransposer (void) // Modül yüklendiğinde çalışan fonksiyon.
    {
        printk (KERN_INFO "Module MatrixTransposer loaded.\n");
 	ProcDirEntry=proc_create (ProcName, 0644, NULL, &FileOperations); // Proc u yaratır.
 	if (ProcDirEntry==NULL) // Proc un yaratılıp yaratılmadığı kontrol eder.
	    {
  	        printk (KERN_INFO "Proc MatrixTransposer did not create.\n");
  		return -ENOMEM;
 	    }
 	printk (KERN_INFO "Proc MatrixTransposer created.\n");
 	return 0;
    }
static void __exit ExitMatrixTransposer (void) // Modül kaldırıldığında çalışan fonksiyon.
    {
        remove_proc_entry (ProcName, NULL); // Proc u kaldırır.
 	printk (KERN_INFO "Proc MatrixTransposer removed.\n");
	printk (KERN_INFO "Module MatrixTransposer removed.\n");
    }
module_init (InitMatrixTransposer);
module_exit (ExitMatrixTransposer);</pre>
	<h3>Makefile</h3>
	<pre>obj-m += MatrixTransposer.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean</pre>
	<br/>
	<li>Modülü yüklemek için c ve Makefile dosyasının bulunduğu dizinde terminali açıp <strong>sudo -s</strong> komutu ile yönetici izini alıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/1.JPG"/>
	<br/>
	<br/>
	<li><strong>make</strong> komutu ile modülü derliyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/2.JPG"/>
	<br/>
	<br/>
	<li><strong>insmod MatrixTransposer.ko m='Satır Sayısı' n='Sütun Sayı'</strong> komutu ile modülü yüklüyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/3.JPG"/>
	<br/>
	<br/>
	<li><strong>dmesg</strong> komutu ile log a bakıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/4.JPG"/>
	<br/>
	<br/>
	<li>Modül başarı ile yüklenmişse aşağıdaki mesajları log a yazar.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/5.JPG"/>
	<br/>
	<br/>
	<li><strong>echo ?,?,?,?,... > /proc/MatrixTransposer</strong> komutu ile inputları aralarına virgül koyarak proc dosyasına yazıyoruz. Bu komut Write fonksiyonunu çalıştırır ve bu fonksiyon girilen inputları formatlı bir şekilde proc dosyasına yazar ve log a yazar.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/6.JPG"/>
	<br/>
	<br/>
	<li><strong>dmesg</strong> komutu ile log a bakıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/7.JPG"/>
	<br/>
	<br/>
	<li>Aşağıda görüldüğü gibi formatlı bir şekilde log a yazılmış.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/8.JPG"/>
	<br/>
	<br/>
	<li><strong>cat /proc/MatrixTransposer</strong> komutu ile proc dosyasını okuyuruz. Bu komut Read fonksiyonunu çalıştırır ve bu fonksiyon proc dosyasından okuduklarını konsol ve log a yazar.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/9.JPG"/>
	<br/>
	<br/>
	<li><strong>dmesg</strong> komutu ile log a bakıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/10.JPG"/>
	<br/>
	<br/>
	<li>Aşağıda görüldüğü gibi log a yazılmış.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/11.JPG"/>
	<br/>
	<br/>
	<li><strong>pico /proc/MatrixTransposer</strong> komutu ile de proc dosyasını okuyabiliriz. Bu komut Read fonksiyonunu çalıştırır ve bu fonksiyon proc dosyasından okuduklarını konsol ve log a yazar.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/12.JPG"/>
	<br/>
	<br/>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/13.JPG"/>
	<br/>
	<br/>
	<li><strong>dmesg</strong> komutu ile log a bakıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/14.JPG"/>
	<br/>
	<br/>
	<li>Aşağıda görüldüğü gibi log a yazılmış.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/15.JPG"/>
	<br/>
	<br/>
	<li><strong>rmmod MatrixTransposer</strong> komutu ile modülü kaldırıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/16.JPG"/>
	<br/>
	<br/>
	<li><strong>dmesg</strong> komutu ile log a bakıyoruz.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/17.JPG"/>
	<br/>
	<br/>
	<li>Modül başarı ile kaldırılmışsa aşağıdaki mesajları log a yazar.</li>
	<img src="https://raw.githubusercontent.com/KeremCanli/KeremCanli.github.io/master/images/MatrixTransposer/18.JPG"/>
	<br/>
	<br/>
</ul>
