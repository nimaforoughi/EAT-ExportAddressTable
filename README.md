# EAT-ExportAddressTable
EAT - Export Address Table diagram with the use of WinDBG

Steps as follow:

1. Retrive the DLL Base address by **lmf**
2. Retreive the Export Directory address by **!dh**
3. Parse the EAT

## EAT Memory Map of a dll/image

![EAT Memory Map](https://github.com/nimaforoughi/EAT-ExportAddressTable/blob/main/Images/EAT-memorymap.png?raw=true)

## How to retreive the DLL base address:

### WinDBG tool:
lmf

![EAT-Export](https://user-images.githubusercontent.com/90676852/174523908-08b571ce-552c-4f7b-bfbd-31eeca43627e.png)

## How to retreive the Export Directory address:

!dh 774e0000

<img src="https://user-images.githubusercontent.com/90676852/174524307-24297a65-4996-47fc-8d6a-3095fbb76d96.png" width="400">


## How to parse the EAT:

dc 774e0000+110340
![EAT2](https://user-images.githubusercontent.com/90676852/174516198-99d8ec8e-1fe5-4cab-823f-18da3d549a79.png)

	typedefstruct_IMAGE_EXPORT_DIRECTORY{
		1-DWORDCharacteristics;
		2-DWORDTimeDateStamp;
		3-WORDMajorVersion;WORDMinorVersion;
		4-DWORDName;// The name of the Dll
		5-DWORDBase;// Number to add to the values found in AddressOfNameOrdinals to retrieve the "real" Ordinal number of the function (by real I mean used to call it by ordinals).
		6-DWORDNumberOfFunctions;// Number of all exported functions      
		7-DWORDNumberOfNames;// Number of functions exported by name-  If this value is 0, then all of the functions in this module are exported by ordinal and none of them is exported by name.     
		8-DWORDAddressOfFunctions;// Export Address Table. Address of the functions addresses array.   
		9-DWORDAddressOfNames;// Export Name table. Address of the functions names array.  a RVA to the list of exported names – it points to an array of NumberOfNames 32-bit values, each being a RVA to the exported symbol name.      
		10-DWORDAddressOfNameOrdinals;// Export sequence number table.  Address of the Ordinals (minus the value of Base) array.    

	}IMAGE_EXPORT_DIRECTORY,*PIMAGE_EXPORT_DIRECTORY;


From <https://alice.climent-pommeret.red/posts/direct-syscalls-hells-halos-syswhispers2/#what-is-a-hook-> .


![EAT2](https://user-images.githubusercontent.com/90676852/174515889-50d40d49-1b22-49ef-938b-6ea4b72a594d.png)

As Alice said:

> 3 fields are super important for us: AddressOfFunctions, AddressOfNames and AddressOfNameOrdinals
> Why ? Because when you call GetProcAddress on a function (let’s say A_SHAFinal) it goes like this:
>
> 1. The string “A_SHAFinal” will be searched in the array AddressOfNames (Export Name table);
> 2. When found, the position in the array is used to find the corresponding Ordinal number in the AddressOfNameOrdinals array (Export sequence number table).
> 3. Finally, the value of the Ordinals retrieved in the AddressOfNameOrdinals array is the index of the address function for A_SHAFinal in AddressOfFunctions (Export Address Table).

	Example: 

	The string A_SHAFinal is at position 0 in AddressOfNames.
	Then to retrieve the ordinal: AddressOfNameOrdinals[0]. 
	The value of AddressOfNameOrdinals[0] is 7.
	To retrieve A_SHAFinal function address we do AddressOfFunctions[7].
	
The full article is presented by Alice Climent Pommeret in <https://alice.climent-pommeret.red/posts/direct-syscalls-hells-halos-syswhispers2/#retrieving-windows-dll-addresses-the-process-environment-block-peb>

## The Diagram of EAT for ntdll32.dll
![EXPORT ADDRESS TABLE](https://github.com/nimaforoughi/EAT-ExportAddressTable/blob/gh-pages/Images/EAT.png?raw=true)




References:

<https://alice.climent-pommeret.red/posts/direct-syscalls-hells-halos-syswhispers2/#retrieving-windows-dll-addresses-the-process-environment-block-peb>

<https://emsea.github.io/2017/12/04/import-by-hash/>
