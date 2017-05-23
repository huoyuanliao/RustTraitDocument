#Rust 基础Trait整理--std::fs

##[std::fs](https://doc.rust-lang.org/stable/std/fs/)

```
	use sys::fs as fs_imp;
	use sys_common::{AsInnerMut, FromInner, AsInner, IntoInner};

	pub struct File {
    	inner: fs_imp::File,
	}

	pub struct Metadata(fs_imp::FileAttr);
	pub struct ReadDir(fs_imp::ReadDir);
	pub struct DirEntry(fs_imp::DirEntry);	
	pub struct OpenOptions(fs_imp::OpenOptions);
	pub struct Permissions(fs_imp::FilePermissions);
	pub struct FileType(fs_imp::FileType);

	pub struct DirBuilder {
    	inner: fs_imp::DirBuilder,
    	recursive: bool,
	}
```

##[File](https://doc.rust-lang.org/stable/std/fs/struct.File.html)

```
	impl File {
	    pub fn open<P: AsRef<Path>>(path: P) -> io::Result<File> 
	    pub fn create<P: AsRef<Path>>(path: P) -> io::Result<File> 
	    pub fn sync_all(&self) -> io::Result<()>
	    pub fn sync_data(&self) -> io::Result<()> 
	    pub fn set_len(&self, size: u64) -> io::Result<()> 
	    pub fn metadata(&self) -> io::Result<Metadata> 
	    pub fn try_clone(&self) -> io::Result<File> 
	    pub fn set_permissions(&self, perm: Permissions) -> io::Result<()> 
	
	}
	impl AsInner<fs_imp::File> for File {
	    fn as_inner(&self) -> &fs_imp::File { &self.inner }
	}
	impl FromInner<fs_imp::File> for File {
	    fn from_inner(f: fs_imp::File) -> File {
	        File { inner: f }
	    }
	}
	impl IntoInner<fs_imp::File> for File {
	    fn into_inner(self) -> fs_imp::File {
	        self.inner
	    }
	}
	
	impl fmt::Debug for File {
	    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
	        self.inner.fmt(f)
	    }
	}
	
	#[stable(feature = "rust1", since = "1.0.0")]
	impl Read for File {
	    fn read(&mut self, buf: &mut [u8]) -> io::Result<usize> {
	        self.inner.read(buf)
	    }
	    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> io::Result<usize> {
	        self.inner.read_to_end(buf)
	    }
	}
	#[stable(feature = "rust1", since = "1.0.0")]
	impl Write for File {
	    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
	        self.inner.write(buf)
	    }
	    fn flush(&mut self) -> io::Result<()> { self.inner.flush() }
	}
	#[stable(feature = "rust1", since = "1.0.0")]
	impl Seek for File {
	    fn seek(&mut self, pos: SeekFrom) -> io::Result<u64> {
	        self.inner.seek(pos)
	    }
	}
	#[stable(feature = "rust1", since = "1.0.0")]
	impl<'a> Read for &'a File {
	    fn read(&mut self, buf: &mut [u8]) -> io::Result<usize> {
	        self.inner.read(buf)
	    }
	    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> io::Result<usize> {
	        self.inner.read_to_end(buf)
	    }
	}
```

##[OpenOptions](https://doc.rust-lang.org/stable/std/fs/struct.OpenOptions.html)

```
	impl OpenOptions {
    	pub fn new() -> OpenOptions {
    	    OpenOptions(fs_imp::OpenOptions::new())
    	}
    	pub fn read(&mut self, read: bool) -> &mut OpenOptions {
    	    self.0.read(read); self
    	}
    	pub fn write(&mut self, write: bool) -> &mut OpenOptions {
     	   self.0.write(write); self
    	}
    	pub fn append(&mut self, append: bool) -> &mut OpenOptions {
     	   self.0.append(append); self
    	}

    	pub fn truncate(&mut self, truncate: bool) -> &mut OpenOptions {
    	    self.0.truncate(truncate); self
    	}
    	pub fn create(&mut self, create: bool) -> &mut OpenOptions {
     	   self.0.create(create); self
    	}
    	pub fn create_new(&mut self, create_new: bool) -> &mut OpenOptions {
    	    self.0.create_new(create_new); self
    	}
	    pub fn open<P: AsRef<Path>>(&self, path: P) -> io::Result<File> {
		     self._open(path.as_ref())
		}
		fn _open(&self, path: &Path) -> io::Result<File> {
			let inner = fs_imp::File::open(path, &self.0)?;
	        Ok(File { inner: inner })
		}
	}

	impl AsInnerMut<fs_imp::OpenOptions> for OpenOptions {
		    fn as_inner_mut(&mut self) -> &mut fs_imp::OpenOptions { &mut self.0 }
		}
```
##[Metadata](https://doc.rust-lang.org/stable/std/fs/struct.Metadata.html)

```
	impl Metadata {
    	pub fn file_type(&self) -> FileType 
    	pub fn is_dir(&self) -> bool { self.file_type().is_dir() }
    	pub fn is_file(&self) -> bool { self.file_type().is_file() }
    	pub fn len(&self) -> u64 { self.0.size() }
    	pub fn permissions(&self) -> Permissions 
	    pub fn modified(&self) -> io::Result<SystemTime> 
    	pub fn accessed(&self) -> io::Result<SystemTime> 
    	pub fn created(&self) -> io::Result<SystemTime> 


	}
	impl fmt::Debug for Metadata {
	    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
	        f.debug_struct("Metadata")
	            .field("file_type", &self.file_type())
	            .field("is_dir", &self.is_dir())
	            .field("is_file", &self.is_file())
	            .field("permissions", &self.permissions())
	            .field("modified", &self.modified())
	            .field("accessed", &self.accessed())
	            .field("created", &self.created())
	            .finish()
	    }
	}
	impl AsInner<fs_imp::FileAttr> for Metadata {
	    fn as_inner(&self) -> &fs_imp::FileAttr { &self.0 }
	}
```

##[ReadDir](https://doc.rust-lang.org/stable/std/fs/struct.ReadDir.html)
```
	impl Iterator for ReadDir {
	    type Item = io::Result<DirEntry>;
	
	    fn next(&mut self) -> Option<io::Result<DirEntry>> {
	        self.0.next().map(|entry| entry.map(DirEntry))
	    }
	}
```

##[DirEntry](https://doc.rust-lang.org/stable/std/fs/struct.DirEntry.html)
```
	impl DirEntry {
	    pub fn path(&self) -> PathBuf { self.0.path() }
	    pub fn metadata(&self) -> io::Result<Metadata> {
	    pub fn file_type(&self) -> io::Result<FileType> {
	    pub fn file_name(&self) -> OsString {
	}
	impl fmt::Debug for DirEntry {
	    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
	        f.debug_tuple("DirEntry")
	            .field(&self.path())
	            .finish()
	    }
	}
	impl AsInner<fs_imp::DirEntry> for DirEntry {
	    fn as_inner(&self) -> &fs_imp::DirEntry { &self.0 }
	}
```

##[DirBuilder](https://doc.rust-lang.org/stable/std/fs/struct.DirBuilder.html)
```
    pub fn new() -> DirBuilder {
    pub fn recursive(&mut self, recursive: bool) -> &mut Self {
    pub fn create<P: AsRef<Path>>(&self, path: P) -> io::Result<()> {
    fn create_dir_all(&self, path: &Path) -> io::Result<()> {
	impl AsInnerMut<fs_imp::DirBuilder> for DirBuilder {
	    fn as_inner_mut(&mut self) -> &mut fs_imp::DirBuilder {
	        &mut self.inner
	    }
}
```

##Common API
```
	pub fn remove_file<P: AsRef<Path>>(path: P) -> io::Result<()> 
	pub fn metadata<P: AsRef<Path>>(path: P) -> io::Result<Metadata> 
	pub fn symlink_metadata<P: AsRef<Path>>(path: P) -> io::Result<Metadata> 
	pub fn rename<P: AsRef<Path>, Q: AsRef<Path>>(from: P, to: Q) -> io::Result<()> 
	pub fn copy<P: AsRef<Path>, Q: AsRef<Path>>(from: P, to: Q) -> io::Result<u64> 
	pub fn hard_link<P: AsRef<Path>, Q: AsRef<Path>>(src: P, dst: Q) -> io::Result<()> 
	pub fn soft_link<P: AsRef<Path>, Q: AsRef<Path>>(src: P, dst: Q) -> io::Result<()> 
	pub fn read_link<P: AsRef<Path>>(path: P) -> io::Result<PathBuf> 
	pub fn canonicalize<P: AsRef<Path>>(path: P) -> io::Result<PathBuf> 
	pub fn create_dir<P: AsRef<Path>>(path: P) -> io::Result<()> 
	pub fn create_dir_all<P: AsRef<Path>>(path: P) -> io::Result<()> 
	pub fn remove_dir<P: AsRef<Path>>(path: P) -> io::Result<()> 
	pub fn remove_dir_all<P: AsRef<Path>>(path: P) -> io::Result<()> 
	pub fn read_dir<P: AsRef<Path>>(path: P) -> io::Result<ReadDir> 
	pub fn set_permissions<P: AsRef<Path>>(path: P, perm: Permissions)
```
