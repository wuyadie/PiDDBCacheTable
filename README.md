# PiDDBCacheTable
Clear your driver from PiDDBCacheTable

```cpp
NTSTATUS ClearPiddbCacheTable(ULONG TimeStamp) {
	//0x140D2D040
	PERESOURCE lock;
	PRTL_AVL_TABLE PiDDBCacheTable;
	if (!NT_SUCCESS(LocatePiDDB(&lock, &PiDDBCacheTable))) {
		kOutput("Locating PiDDB Failed\n");
		return STATUS_NOT_FOUND;
	}

	//Found PiDDBCacheTable
	kOutput("[-] PiDDBCacheTable -> 0x%llx\n", PiDDBCacheTable);

	//Check Entries
	kOutput("[-] PiDDBCacheTable->NumberGenericTableElements -> %d\n", PiDDBCacheTable->NumberGenericTableElements);

	//Get First Entry
	PiDDBCacheEntry* FirstEntry = PiDDBCacheTable->BalancedRoot.RightChild;


	kOutput("[-] First Entry -> 0x%p\t:\tFirst Entry TimeStamp -> 0x%x\n",
		FirstEntry,
		//FirstEntry->DriverName,
		FirstEntry->TimeDateStamp
	);

	for (PiDDBCacheEntry* CurrEntry = 
		(PiDDBCacheEntry*)RtlEnumerateGenericTableAvl(PiDDBCacheTable, TRUE);				/* restart */
		CurrEntry != NULL;																	/* as long as the current entry is valid */
		CurrEntry = (PiDDBCacheEntry*)RtlEnumerateGenericTableAvl(PiDDBCacheTable, FALSE)	/* no restart, get latest element */
		) 
	{

		ExAcquireResourceExclusiveLite(&lock, TRUE);

		if (CurrEntry->TimeDateStamp == TimeStamp) {
			kOutput("[-] Entry Name -> %wZ\t:\tEntry TimeStamp -> 0x%x\n",
				CurrEntry->DriverName,
				CurrEntry->TimeDateStamp
			);

			//Unlink
			RemoveEntryList(&CurrEntry->List);
			//Delete
			RtlDeleteElementGenericTableAvl(&PiDDBCacheTable, CurrEntry);
			//Clean up
			ExReleaseResourceLite(PiDDBCacheTable);

			return STATUS_SUCCESS;
		}
	}
	ExReleaseResourceLite(PiDDBCacheTable);
	return STATUS_NOT_FOUND;
}```
