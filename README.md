# neocorestore
A __WIP__, experimental take on Corestore.

These features might eventually become Corestore v6:
* Replaces derived-key-storage with a Hyperbee database of names.
* The API is now very name-centric. Every `get` operation must either provide a key or a name.
* No more `default` -- keep everything named.
* Namespacing is far more lightweight.
* Reference counting is far simpler, and is decoupled from namespaces.
* Backup/restore write capabilities from a "manifest" file.



