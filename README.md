# virtual

## 虚函数表
[请找出下面哪句话不对]
对C++ 了解的人都应该知道虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。简称为V-Table。在这个表中，主要是一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其内容真实反应实际的函数。这样，在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得尤为重要了，它就像一个地图一样，指明了实际所应该调用的函数。

C++的编译器应该是保证虚函数表的指针存在于对象实例中最前面的位置（这是为了保证取到虚函数表的有最高的性能——如果有多层继承或是多重继承的情况下）。 这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。

举个栗子
```

class Base {
     public:
            virtual void f() { cout << "Base::f" << endl; }
            virtual void g() { cout << "Base::g" << endl; }
            virtual void h() { cout << "Base::h" << endl; }
};

int main() {

    typedef void(*Fun)(void);
    Base b;
    Fun pFun = NULL;
    cout << "虚函数表地址：" << (int*)*(int*)(&b) << endl;
    cout << "虚函数表 — 第一个函数地址：" << (int*)*(int*)*(int*)(&b) << endl;
    // Invoke(调用) the first virtual function
    pFun = (Fun)*((int*)*(int*)(&b));
    pFun();
}

```
插播一则函数指针，直接看例子
```
# include <stdio.h>
int Max(int, int);  //函数声明
int main(void)
{
    int(*p)(int, int);  //定义一个函数指针
    int a, b, c;
    p = Max;  //把函数Max赋给指针变量p, 使p指向Max函数
    printf("please enter a and b:");
    scanf("%d%d", &a, &b);
    c = (*p)(a, b);  //通过函数指针调用Max函数
    printf("a = %d\nb = %d\nmax = %d\n", a, b, c);
    return 0;
}
int Max(int x, int y)  //定义Max函数
{
    int z;
    if (x > y)
    {
        z = x;
    }
    else
    {
        z = y;
    }
    return z;
}
```

回到第一个例子，对(int*)*(int*)*(int*)(&b)进行分析一下：对&b强转为int*得到了b的内存首地址，然后(int*)*是对此地址取值再强转得到虚函数表的首地址，后面再一次取值强转就得到了一个虚函数的地址指针
[图1]

### 一般继承（无虚函数覆盖）
[继承关系的图]

对于实例：Derive d; 它所指向的虚函数表如下
[图]

可以看到：
* 虚函数按照其声明顺序放于表中
* 父类的虚函数在子类的虚函数前面

举个栗子

### 一般继承（有虚函数覆盖）
[图]

可以看到:
* 覆盖的f()函数被放到了虚表中原来父类虚函数的位置
* 没有被覆盖的函数依旧

这样，我们就可以看到对于下面这样的程序.由b所指的内存中的虚函数表的f()的位置已经被Derive::f()函数地址所取代，于是在实际调用发生时，是Derive::f()被调用了
```
Base *b = new Derive();
b->f(); 
```

### 多继承（无虚函数覆盖）
[图]
子类的虚函数表，是下面这个样子
[图]
可以看到：
* 每个父类都有自己的虚表
* 子类的成员函数被放到了第一个父类的表中。（所谓的第一个父类是按照声明顺序来判断的）
这样做就是为了解决不同的父类类型的指针指向同一个子类实例，而能够调用到实际的函数。

### 多继承（有虚函数覆盖）
[图]
子类的虚函数表，是下面这个样子
[图]
我们可以看见，三个父类虚函数表中的f()的位置被替换成了子类的函数指针.
举个栗子

### 虚函数表和虚函数在内存中的位置
举个栗子
test3.cpp
执行后输出
```
addr of vb : 4197344
addr of vfun : 4197098
vfun called!
```
用objdump -s 可以解析ELF格式的可执行文件中的分段信息
```

a.out:     file format elf64-x86-64

Contents of section .interp:
 400238 2f6c6962 36342f6c 642d6c69 6e75782d  /lib64/ld-linux-
 400248 7838362d 36342e73 6f2e3200           x86-64.so.2.    
Contents of section .note.ABI-tag:
 400254 04000000 10000000 01000000 474e5500  ............GNU.
 400264 00000000 02000000 06000000 20000000  ............ ...
Contents of section .note.gnu.build-id:
 400274 04000000 14000000 03000000 474e5500  ............GNU.
 400284 093a23d4 5c23f73a 8d035c16 0a13a0bf  .:#.\#.:..\.....
 400294 7b535378                             {SSx            
Contents of section .gnu.hash:
 400298 03000000 0d000000 01000000 06000000  ................
 4002a8 80011000 01011000 0d000000 0e000000  ................
 4002b8 0f000000 c9810ad2 21fdf409 2845d54c  ........!...(E.L
 4002c8 15980c43                             ...C            
Contents of section .dynsym:
 4002d0 00000000 00000000 00000000 00000000  ................
 4002e0 00000000 00000000 5a010000 12000000  ........Z.......
 4002f0 00000000 00000000 00000000 00000000  ................
 400300 10000000 20000000 00000000 00000000  .... ...........
 400310 00000000 00000000 1f000000 20000000  ............ ...
 400320 00000000 00000000 00000000 00000000  ................
 400330 b8000000 12000000 00000000 00000000  ................
 400340 00000000 00000000 fa000000 12000000  ................
 400350 00000000 00000000 00000000 00000000  ................
 400360 7b010000 12000000 00000000 00000000  {...............
 400370 00000000 00000000 6e010000 12000000  ........n.......
 400380 00000000 00000000 00000000 00000000  ................
 400390 33000000 20000000 00000000 00000000  3... ...........
 4003a0 00000000 00000000 18010000 12000000  ................
 4003b0 00000000 00000000 00000000 00000000  ................
 4003c0 4f000000 20000000 00000000 00000000  O... ...........
 4003d0 00000000 00000000 a7000000 12000000  ................
 4003e0 00000000 00000000 00000000 00000000  ................
 4003f0 12010000 12000000 00000000 00000000  ................
 400400 00000000 00000000 69000000 21001a00  ........i...!...
 400410 80106000 00000000 58000000 00000000  ..`.....X.......
 400420 bf000000 12000000 80084000 00000000  ..........@.....
 400430 00000000 00000000 8f000000 12000000  ................
 400440 50084000 00000000 00000000 00000000  P.@.............
 400450 50010000 11001a00 e0106000 00000000  P.........`.....
 400460 10010000 00000000                    ........        
Contents of section .dynstr:
 400468 006c6962 73746463 2b2b2e73 6f2e3600  .libstdc++.so.6.
 400478 5f5f676d 6f6e5f73 74617274 5f5f005f  __gmon_start__._
 400488 4a765f52 65676973 74657243 6c617373  Jv_RegisterClass
 400498 6573005f 49544d5f 64657265 67697374  es._ITM_deregist
 4004a8 6572544d 436c6f6e 65546162 6c65005f  erTMCloneTable._
 4004b8 49544d5f 72656769 73746572 544d436c  ITM_registerTMCl
 4004c8 6f6e6554 61626c65 005f5a54 564e3130  oneTable._ZTVN10
 4004d8 5f5f6378 78616269 76313137 5f5f636c  __cxxabiv117__cl
 4004e8 6173735f 74797065 5f696e66 6f45005f  ass_type_infoE._
 4004f8 5a4e5374 38696f73 5f626173 6534496e  ZNSt8ios_base4In
 400508 69744431 4576005f 5a4e536f 6c734550  itD1Ev._ZNSolsEP
 400518 4652536f 535f4500 5f5a646c 5076005f  FRSoS_E._ZdlPv._
 400528 5a537434 656e646c 49635374 31316368  ZSt4endlIcSt11ch
 400538 61725f74 72616974 73496345 45525374  ar_traitsIcEERSt
 400548 31336261 7369635f 6f737472 65616d49  13basic_ostreamI
 400558 545f5430 5f455336 5f005f5a 4e537438  T_T0_ES6_._ZNSt8
 400568 696f735f 62617365 34496e69 74433145  ios_base4InitC1E
 400578 76005f5a 6e776d00 5f5a5374 6c734953  v._Znwm._ZStlsIS
 400588 74313163 6861725f 74726169 74734963  t11char_traitsIc
 400598 45455253 74313362 61736963 5f6f7374  EERSt13basic_ost
 4005a8 7265616d 4963545f 4553355f 504b6300  reamIcT_ES5_PKc.
 4005b8 5f5a5374 34636f75 74005f5a 4e536f6c  _ZSt4cout._ZNSol
 4005c8 73456c00 6c696263 2e736f2e 36005f5f  sEl.libc.so.6.__
 4005d8 6378615f 61746578 6974005f 5f6c6962  cxa_atexit.__lib
 4005e8 635f7374 6172745f 6d61696e 00474c49  c_start_main.GLI
 4005f8 42435f32 2e322e35 00435858 4142495f  BC_2.2.5.CXXABI_
 400608 312e3300 474c4942 4358585f 332e3400  1.3.GLIBCXX_3.4.
Contents of section .gnu.version:
 400618 00000200 00000000 02000200 03000300  ................
 400628 00000200 00000200 02000400 02000200  ................
 400638 0200                                 ..              
Contents of section .gnu.version_r:
 400640 01000100 64010000 10000000 20000000  ....d....... ...
 400650 751a6909 00000300 8d010000 00000000  u.i.............
 400660 01000200 01000000 10000000 00000000  ................
 400670 d3af6b05 00000400 99010000 10000000  ..k.............
 400680 74299208 00000200 a4010000 00000000  t)..............
Contents of section .rela.dyn:
 400690 f80f6000 00000000 06000000 02000000  ..`.............
 4006a0 00000000 00000000 80106000 00000000  ..........`.....
 4006b0 05000000 0d000000 00000000 00000000  ................
 4006c0 e0106000 00000000 05000000 10000000  ..`.............
 4006d0 00000000 00000000                    ........        
Contents of section .rela.plt:
 4006d8 18106000 00000000 07000000 01000000  ..`.............
 4006e8 00000000 00000000 20106000 00000000  ........ .`.....
 4006f8 07000000 04000000 00000000 00000000  ................
 400708 28106000 00000000 07000000 05000000  (.`.............
 400718 00000000 00000000 30106000 00000000  ........0.`.....
 400728 07000000 06000000 00000000 00000000  ................
 400738 38106000 00000000 07000000 07000000  8.`.............
 400748 00000000 00000000 40106000 00000000  ........@.`.....
 400758 07000000 0f000000 00000000 00000000  ................
 400768 48106000 00000000 07000000 09000000  H.`.............
 400778 00000000 00000000 50106000 00000000  ........P.`.....
 400788 07000000 0b000000 00000000 00000000  ................
 400798 58106000 00000000 07000000 0e000000  X.`.............
 4007a8 00000000 00000000 60106000 00000000  ........`.`.....
 4007b8 07000000 0c000000 00000000 00000000  ................
Contents of section .init:
 4007c8 4883ec08 488b0525 08200048 85c07405  H...H..%. .H..t.
 4007d8 e8c30000 004883c4 08c3               .....H....      
Contents of section .plt:
 4007f0 ff351208 2000ff25 14082000 0f1f4000  .5.. ..%.. ...@.
 400800 ff251208 20006800 000000e9 e0ffffff  .%.. .h.........
 400810 ff250a08 20006801 000000e9 d0ffffff  .%.. .h.........
 400820 ff250208 20006802 000000e9 c0ffffff  .%.. .h.........
 400830 ff25fa07 20006803 000000e9 b0ffffff  .%.. .h.........
 400840 ff25f207 20006804 000000e9 a0ffffff  .%.. .h.........
 400850 ff25ea07 20006805 000000e9 90ffffff  .%.. .h.........
 400860 ff25e207 20006806 000000e9 80ffffff  .%.. .h.........
 400870 ff25da07 20006807 000000e9 70ffffff  .%.. .h.....p...
 400880 ff25d207 20006808 000000e9 60ffffff  .%.. .h.....`...
 400890 ff25ca07 20006809 000000e9 50ffffff  .%.. .h.....P...
Contents of section .plt.got:
 4008a0 ff255207 20006690                    .%R. .f.        
Contents of section .text:
 4008b0 31ed4989 d15e4889 e24883e4 f0505449  1.I..^H..H...PTI
 4008c0 c7c0900b 400048c7 c1200b40 0048c7c7  ....@.H.. .@.H..
 4008d0 a6094000 e857ffff fff4660f 1f440000  ..@..W....f..D..
 4008e0 b87f1060 0055482d 78106000 4883f80e  ...`.UH-x.`.H...
 4008f0 4889e576 1bb80000 00004885 c074115d  H..v......H..t.]
 400900 bf781060 00ffe066 0f1f8400 00000000  .x.`...f........
 400910 5dc30f1f 4000662e 0f1f8400 00000000  ]...@.f.........
 400920 be781060 00554881 ee781060 0048c1fe  .x.`.UH..x.`.H..
 400930 034889e5 4889f048 c1e83f48 01c648d1  .H..H..H..?H..H.
 400940 fe7415b8 00000000 4885c074 0b5dbf78  .t......H..t.].x
 400950 106000ff e00f1f00 5dc3660f 1f440000  .`......].f..D..
 400960 803d8908 20000075 11554889 e5e86eff  .=.. ..u.UH...n.
 400970 ffff5dc6 05760820 0001f3c3 0f1f4000  ..]..v. ......@.
 400980 bf100e60 0048833f 007505eb 930f1f00  ...`.H.?.u......
 400990 b8000000 004885c0 74f15548 89e5ffd0  .....H..t.UH....
 4009a0 5de97aff ffff5548 89e55348 83ec28bf  ].z...UH..SH..(.
 4009b0 08000000 e8d7feff ff4889c3 4889dfe8  .........H..H...
 4009c0 f6000000 48895dd8 488b45d8 8b004898  ....H.].H.E...H.
 4009d0 488945e0 488b45e0 8b004898 488945e8  H.E.H.E...H.H.E.
 4009e0 beb10b40 00bfe010 6000e871 feffff48  ...@....`..q...H
 4009f0 89c2488b 45e04889 c64889d7 e8fffdff  ..H.E.H..H......
 400a00 ffbe8008 40004889 c7e862fe ffffbebf  ....@.H...b.....
 400a10 0b4000bf e0106000 e843feff ff4889c2  .@....`..C...H..
 400a20 488b45e8 4889c648 89d7e8d1 fdffffbe  H.E.H..H........
 400a30 80084000 4889c7e8 34feffff 488b45e8  ..@.H...4...H.E.
 400a40 ffd0488b 5dd84885 db741048 89dfe87f  ..H.].H..t.H....
 400a50 00000048 89dfe8b5 fdffffb8 00000000  ...H............
 400a60 4883c428 5b5dc355 4889e548 83ec1089  H..([].UH..H....
 400a70 7dfc8975 f8837dfc 01752781 7df8ffff  }..u..}..u'.}...
 400a80 0000751e bff11160 00e892fd ffffba70  ..u....`.......p
 400a90 106000be f1116000 bf500840 00e89efd  .`....`..P.@....
 400aa0 ffff90c9 c3554889 e5beffff 0000bf01  .....UH.........
 400ab0 000000e8 afffffff 5dc35548 89e54889  ........].UH..H.
 400ac0 7df8bae0 0b400048 8b45f848 8910905d  }....@.H.E.H...]
 400ad0 c3905548 89e54889 7df8bae0 0b400048  ..UH..H.}....@.H
 400ae0 8b45f848 8910905d c3905548 89e54883  .E.H...]..UH..H.
 400af0 ec104889 7df8bea4 0b4000bf e0106000  ..H.}....@....`.
 400b00 e85bfdff ffbe8008 40004889 c7e85efd  .[......@.H...^.
 400b10 ffff90c9 c3662e0f 1f840000 00000090  .....f..........
 400b20 41574156 4189ff41 5541544c 8d25c602  AWAVA..AUATL.%..
 400b30 20005548 8d2dce02 20005349 89f64989   .UH.-.. .SI..I.
 400b40 d54c29e5 4883ec08 48c1fd03 e877fcff  .L).H...H....w..
 400b50 ff4885ed 742031db 0f1f8400 00000000  .H..t 1.........
 400b60 4c89ea4c 89f64489 ff41ff14 dc4883c3  L..L..D..A...H..
 400b70 014839eb 75ea4883 c4085b5d 415c415d  .H9.u.H...[]A\A]
 400b80 415e415f c390662e 0f1f8400 00000000  A^A_..f.........
 400b90 f3c3                                 ..              
Contents of section .fini:
 400b94 4883ec08 4883c408 c3                 H...H....       
Contents of section .rodata:
 400ba0 01000200 7666756e 2063616c 6c656421  ....vfun called!
 400bb0 00616464 72206f66 20766220 3a200061  .addr of vb : .a
 400bc0 64647220 6f662076 66756e20 3a200000  ddr of vfun : ..
 400bd0 00000000 00000000 e80b4000 00000000  ..........@.....
 400be0 ea0a4000 00000000 90106000 00000000  ..@.......`.....
 400bf0 f80b4000 00000000 314100             ..@.....1A.     
Contents of section .eh_frame_hdr:
 400bfc 011b033b 58000000 0a000000 f4fbffff  ...;X...........
 400c0c a4000000 b4fcffff 74000000 aafdffff  ........t.......
 400c1c 2c010000 6bfeffff 54010000 a9feffff  ,...k...T.......
 400c2c 74010000 befeffff cc000000 d6feffff  t...............
 400c3c ec000000 eefeffff 0c010000 24ffffff  ............$...
 400c4c 94010000 94ffffff dc010000           ............    
Contents of section .eh_frame:
 400c58 14000000 00000000 017a5200 01781001  .........zR..x..
 400c68 1b0c0708 90010710 14000000 1c000000  ................
 400c78 38fcffff 2a000000 00000000 00000000  8...*...........
 400c88 14000000 00000000 017a5200 01781001  .........zR..x..
 400c98 1b0c0708 90010000 24000000 1c000000  ........$.......
 400ca8 48fbffff b0000000 000e1046 0e184a0f  H..........F..J.
 400cb8 0b770880 003f1a3b 2a332422 00000000  .w...?.;*3$"....
 400cc8 1c000000 44000000 eafdffff 17000000  ....D...........
 400cd8 00410e10 8602430d 06520c07 08000000  .A....C..R......
 400ce8 1c000000 64000000 e2fdffff 17000000  ....d...........
 400cf8 00410e10 8602430d 06520c07 08000000  .A....C..R......
 400d08 1c000000 84000000 dafdffff 2b000000  ............+...
 400d18 00410e10 8602430d 06660c07 08000000  .A....C..f......
 400d28 24000000 a4000000 76fcffff c1000000  $.......v.......
 400d38 00410e10 8602430d 06458303 02b70c07  .A....C..E......
 400d48 08000000 00000000 1c000000 cc000000  ................
 400d58 0ffdffff 3e000000 00410e10 8602430d  ....>....A....C.
 400d68 06790c07 08000000 1c000000 ec000000  .y..............
 400d78 2dfdffff 15000000 00410e10 8602430d  -........A....C.
 400d88 06500c07 08000000 44000000 0c010000  .P......D.......
 400d98 88fdffff 65000000 00420e10 8f02420e  ....e....B....B.
 400da8 188e0345 0e208d04 420e288c 05480e30  ...E. ..B.(..H.0
 400db8 8606480e 3883074d 0e40720e 38410e30  ..H.8..M.@r.8A.0
 400dc8 410e2842 0e20420e 18420e10 420e0800  A.(B. B..B..B...
 400dd8 14000000 54010000 b0fdffff 02000000  ....T...........
 400de8 00000000 00000000 00000000           ............    
Contents of section .init_array:
 600df8 80094000 00000000 a50a4000 00000000  ..@.......@.....
Contents of section .fini_array:
 600e08 60094000 00000000                    `.@.....        
Contents of section .jcr:
 600e10 00000000 00000000                    ........        
Contents of section .dynamic:
 600e18 01000000 00000000 01000000 00000000  ................
 600e28 01000000 00000000 64010000 00000000  ........d.......
 600e38 0c000000 00000000 c8074000 00000000  ..........@.....
 600e48 0d000000 00000000 940b4000 00000000  ..........@.....
 600e58 19000000 00000000 f80d6000 00000000  ..........`.....
 600e68 1b000000 00000000 10000000 00000000  ................
 600e78 1a000000 00000000 080e6000 00000000  ..........`.....
 600e88 1c000000 00000000 08000000 00000000  ................
 600e98 f5feff6f 00000000 98024000 00000000  ...o......@.....
 600ea8 05000000 00000000 68044000 00000000  ........h.@.....
 600eb8 06000000 00000000 d0024000 00000000  ..........@.....
 600ec8 0a000000 00000000 b0010000 00000000  ................
 600ed8 0b000000 00000000 18000000 00000000  ................
 600ee8 15000000 00000000 00000000 00000000  ................
 600ef8 03000000 00000000 00106000 00000000  ..........`.....
 600f08 02000000 00000000 f0000000 00000000  ................
 600f18 14000000 00000000 07000000 00000000  ................
 600f28 17000000 00000000 d8064000 00000000  ..........@.....
 600f38 07000000 00000000 90064000 00000000  ..........@.....
 600f48 08000000 00000000 48000000 00000000  ........H.......
 600f58 09000000 00000000 18000000 00000000  ................
 600f68 feffff6f 00000000 40064000 00000000  ...o....@.@.....
 600f78 ffffff6f 00000000 02000000 00000000  ...o............
 600f88 f0ffff6f 00000000 18064000 00000000  ...o......@.....
 600f98 00000000 00000000 00000000 00000000  ................
 600fa8 00000000 00000000 00000000 00000000  ................
 600fb8 00000000 00000000 00000000 00000000  ................
 600fc8 00000000 00000000 00000000 00000000  ................
 600fd8 00000000 00000000 00000000 00000000  ................
 600fe8 00000000 00000000 00000000 00000000  ................
Contents of section .got:
 600ff8 00000000 00000000                    ........        
Contents of section .got.plt:
 601000 180e6000 00000000 00000000 00000000  ..`.............
 601010 00000000 00000000 06084000 00000000  ..........@.....
 601020 16084000 00000000 26084000 00000000  ..@.....&.@.....
 601030 36084000 00000000 46084000 00000000  6.@.....F.@.....
 601040 56084000 00000000 66084000 00000000  V.@.....f.@.....
 601050 76084000 00000000 86084000 00000000  v.@.......@.....
 601060 96084000 00000000                    ..@.....        
Contents of section .data:
 601068 00000000 00000000 00000000 00000000  ................
Contents of section .comment:
 0000 4743433a 20285562 756e7475 20352e34  GCC: (Ubuntu 5.4
 0010 2e302d36 7562756e 7475317e 31362e30  .0-6ubuntu1~16.0
 0020 342e3132 2920352e 342e3020 32303136  4.12) 5.4.0 2016
 0030 30363039 00                          0609.           
```

对地址进制转换： https://tool.oschina.net/hexconvert/
得到虚函数表的地址为400be0:在.rodata这个段
虚函数的地址为400aea:.text代码段
综上所述： C++中虚函数表位于只读数据段（.rodata），也就是C++内存模型中的常量区；而虚函数则位于代码段（.text），也就是C++内存模型中的代码区。如下图
图[https://www.cnblogs.com/senior-engineer/p/7832915.html]
