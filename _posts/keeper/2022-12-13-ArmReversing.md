---
published: true
title:  "KEEPER - ArmReversing 스터디"
categories:
  - keeper
---

ARM Reversing의 기초 지식을 함께 공유하는 스터디입니다.

기본적인 리버싱 개념을 배웠고 배운 개념을 토대로 팀을 구성해 스터디장님이 주신 ARM 리버싱 문제를 풀어보고 발표했습니다.

## ELF 파일
ELF(Executable and Linkable Format)
실행 파일, 목적 파일, 공유 라이브러리 그리고 코어 덤프를 위한 표준 파일 형식입니다.

ELF에서는 프로그램이 실행될 때 메모리에 올라가야 할 각각의 부분들을 미리 정리하여 관리하다가 실행을 하게 되면 코드, 전역데이터. 읽기 전용 데이터 등등을 메모리에 올리게 됩니다.

이렇게 메모리에 올라온 주소 공간의 .text섹션(코드 영역)의 명령어를 한 줄씩 실행하며 프로세스가 진행됩니다.

## readelf
readelf : ELF 파일 정보를 확인 할 수 있는 도구

형식 : `readelf [옵션] elf파일`

[ELF 헤더 출력]
|옵션|내용|
|---|---|
-h|ELF 파일 헤더|
-l|프로그램 헤더|
-S|섹션 헤더|
-e|위 세가지 모두|


[ ELF 정보 출력 ]
|옵션|내용|
|---|---|
-s|심볼테이블|
-r|재배치 정보|
-d|동적 세그먼트|
-V|버전 정보|
-A|아키텍처 의존 정보|
-a|모든 헤더 및 정보|
-n|코어 노트|
-u|unwind 정보|

## ELF 파일 형식
ELF 의 Header에 표시 되어있습니다.

`readelf -h [elf파일]`  or `readelf -a [elf파일]`로 확인 가능!

1. ET_NONE (ELF type none) (0x0000)
    : 알 수 없는 형식 → 아직 정의되지 않았거나 알 수 없다는 의미
    
2. ET_REL (ELF type relocatable) (0x0001) - 링킹 가능한 포맷
    : 재배열이 가능한 파일 형식 → 아직 실행파일에 링킹되지 않은 상태
    
3. ET_EXEC (ELF type executable) (0x0002) - 실행 가능한 포맷
    : 실행 파일 형식(=프로그램) → 프로세스의 시작 지점인 엔트리 포인트가 존재
    
4. ET_DYN (ELF type dynamic) (0x0003) - 실행 가능하면서 링킹 가능한 포맷
    : 공유 오브젝트 파일 형식 → 동적 링킹이 가능한 오브젝트 파일 (=공유 라이브러리)
    
5. ET_CORE (ELF type core) (0x0004)
    : 코어 파일 형식 → 프로세스 이미지의 전체 덤프
      주로 프로그램이 비정상 종료되거나 SIGSEGV 시그널(세그멘테이션 폴트)로 인해 프로세스가 종료된 경우 생성


## ELF 파일의 구성
Linking View : relocatable file의 형식 
(link 하기 전의 목적파일(*.o)은 linking view)
나중에 link 과정을 위해 여러가지 정보를 심어놓은 것

Excution View : link가 끝난 후 완전히 실행 가능한 형태가 된 ELF 형식

![핵심to부가](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/armReversing/reversing.png?raw=true)

## ELF Header
파일의 구성을 나타내는 로드맵 같은 역할

linking view 이던 execution view 이던 무조건 가지고 있음
progream header table 과 section header table의 위치를 결정함
zero offset에서 시작.

~~~c
#define EI_NIDENT 16
typedef struct {
    unsigned char e_ident[EI_NIDENT];
		uint16_t e_type;     // ELF 파일 형식 (위에서 언급)
		uint16_t e_machine;    // CPU 아키텍쳐 타입
		uint32_t e_version;    // ELF 파일 버전 (거의 무조건 0x1)
		ElfN_Addr e_entry;    // Virtual Memory상의 엔트리 포인트
		ElfN_Off e_phoff;      // program header table 파일 오프셋
		ElfN_Off e_shoff;      // section header table 파일 오프셋
		uint32_t e_flags;      // processor specific flags
		uint16_t e_ehsize;    // ELF header 사이즈 (bytes)
		uint16_t e_phentsize;    // program header table entry 사이즈
		uint16_t e_phnum;    // program header table entry의 갯수
		uint16_t e_shentsize;    // section header table entry 사이즈
		uint16_t e_shnum;    // section header table entry의 갯수
		uint16_t e_shstrndx;    // section header string table의 인덱스
}ElfN_Ehdr;
~~~

e_ident : 파일의 내용을 해석하고 디코딩하기 위해 필요한 정보를 담고 있음

0x00 (4bytes) - 7F 45 4C 46 (ELF) : ELF 파일의 Magic Number

0x04 (1bytes) e_ident[EI_CLASS] : 32bit format인 경우 1, 64bit format인 경우 2를 세팅.

0x05 (1bytes) e_ident[EI_DATA] : Little Endian인 경우 1, Big Endian인 경우 2를 세팅

0x06 (1bytes) e_ident[EI_VERSION] : ELF 파일의 버전, 1로 세팅

0x07 (1bytes) e_ident[EI_OSABI] : OS 식별자

0x08 (1bytes) e_ident[EI_ABIVERSION]

0x09 (7bytes) e_ident[EI_PAD] : 사용하지 않음. - 0x00으로 채움.

## 세그먼트
여러 linkable 파일들의 섹션들을 링커가 돌면서 통합해주고, 이것들에 대한 정보를 program header에 있음.

![핵심to부가](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/armReversing/seg.png?raw=true)

이 테이블에는 세그먼트의 타입, 파일 오프셋, 물리주소, 가상주소, 파일크기, 메모리이미지크기, 정렬방식 등이 정의되어 있음.

구조체
~~~c
typedef strcut {
		uint32_t     p_type;        // 세그먼트 형식
		Elf32_Off    p_offset;      // 세그먼트 오프셋
		Elf32_Addr    p_vaddr;    // 세그먼트 가상 주소
		Elf32_Addr    p_paddr;    // 세그먼트 물리 주소
		uint32_t     p_filesz;        // 파일에서의 세그먼트 크기
		uint32_t     p_memsz;     // 메모리에서의 세그먼트 크기
		uint32_t     p_flags;        // 세그먼트 플래그 (실행 | 읽기 | 쓰기)
		uint32_t     p_align;        // 메모리에서의 세그먼트 정렬
}Elf32_Phdr
~~~

## Section Header Table
프로그램 실행에 필수 요구 정보는 아님
프로그램 헤더 테이블이 프로그램 메모리 레이아웃을 정의하고, 섹션 헤더는 이를 보완해주는 역할

섹션은 세그먼트의 구성요소라고 할 수 있다.

### 섹션의 종류

1.  .text 섹션
    
    코드 섹션으로 프로그램 코드 명령어가 저장된다.
    
    SHT_PROGBITS 섹션 형식을 따르며, 텍스트 세그먼트에 존재한다.
    
2.  .rodata 섹션
    
    읽기 전용 데이터 섹션으로 문자열 등이 저장된다.
    
    이 섹션은 읽기 전용이므로 반드시 읽기 전용 세그먼트에 존재해야 한다.
    ( 따라서 데이터 세그먼트가 아닌 텍스트 세그먼트에 위치 할 때도 있다.)
    
    SHT_PROGBITS 섹션 형식을 따른다.
    
3.  .plt 섹션 (Procedure Linkage Table)
    
    동적 링커가 공유 라이브러리에서 import한 함수를 호출하기 위한 정보가 담긴다.
    
    이 정보에는 코드가 포함되어 있다. 따라서 텍스트 세그먼트에 위치한다.
    
    SHT_PROGBITS 섹션 형식을 따른다.
    
4.  .data 섹션
    
    초기화된 전역 변수 등의 데이터를 저장한다. 따라서 데이터 세그먼트에 존재한다.
    
    SHT_PROGBITS 섹션 형식을 따른다.
    
5.  .bss 섹션
    
    초기화 되지 않은 전역 데이터를 저장한다. 따라서 데이터 세그먼트에 존재한다
    
    디스크 상에서는 섹션의 존재를 나타내기 위해 4바이트만 크기를 차지하지만, 프로그램이 로드될 때 원래의 크기를 갖고 0으로 초기화된 값으로 채워져 할당/정렬되어 실행된다.
    
    실제 데이터를 가지지 않아 SHT_NOBITS 섹션 형식을 가진다.
    
6.  .got.plt 섹션 (Global Offset Table)
    
    이 섹션과 PLT 정보를 통해 import된 공유 라이브러리 함수에 접근이 가능하다.
    
    동적 링커에 의해 런터임에 정보가 수정될 수 있다.
    
    got overwrite 라는 익스플로잇 공격이 이 섹션의 주소를 덮어쓰는 공격이다.
    
    프로그램 실행과 관련이 있으므로 SHT_PROGBITS 섹션 형식을 가진다.
    
7.  .dynsym 섹션
    
    공유 라이브러리로부터 import된 동적 **심볼** 정보를 담고 있다.
    
    이 섹션은 텍스크 세그먼트에 존재한다.
    
    SHT_PROGBITS 섹션 형식을 따른다.
    
8.  .dynstr 섹션
    
    동적 심볼 문자열 테이블 정보를 담고 있다.
    
9.  .rel.* 섹션
    
    재배열 섹션은 ELF 객체 또는 프로세스 이미지의 어떤 부분이 링킹 시(여러 오브젝트 파일을 링킹할 때) or 런타임에서 재배열 되어야 하는지 알려준다. -> 즉 위치 오프셋
    
    재배열 데이터가 내부에 포함되며, SHT_REL 섹션 형식을 가진다.
    
10.  .hash 섹션 (= .gnu.hash 섹션)
    
    심볼 참조 해시 테이블
    
    리눅스 ELF에서 사용하는 심볼 이름 참조 코드이다.
    
11.  .symtab 섹션
    
    ElfN_Syn 형식의 심볼 정보를 담고 있다.
    
    SHT_SYMTAB 섹션 형식을 가진다.
    
12.  .strtab 섹션
    
    .symtab 섹션의 ElfN_Sym 구조체에 존재하는 st_name 엔트리에 의해 참조되는 심볼 문자열 테이블 정보를 담고 있다.
    
    문자열 테이블이 위치하기 때문에 SHT_STRTAB 섹션 형식을 갖는다.
    
13.  .shstrtab 섹션
    
    null 종료 문자열과 각 섹션의 이름을 나타내는 섹션 헤더이다.
    
    문자열 테이블을 담고 있다.
    
    ELF 파일 헤더의 e_shstrndx포인터 & .shstrtab의 오프셋에 의해 참조된다.
    
    문자열 테이블을 가지므로 SHT_STRTAB 섹션 형식을 가진다.
    
14.  .ctors / .dtors 섹션 ( Constructors, 생성자 / Destructors, 소멸자)
    
    초기화 및 종료 함수 포인터를 포함한다. -> 각각은 main() 함수 호출 이전 / 이후에 실행이 된다.
    
    안티 디버깅에 자주 활용이 된다.
