# Snap issue with epoll

With Ruby installed, build and run snap with:

```
bundle
bundle exec rake snap
sudo snap install --dangerous async-issue-360_0.1.0_amd64.snap
async-issue-360
```

On `core22` and `core24`, this will fail with:

```
/snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:383:in `select': Operation not permitted - select_internal_with_gvl:epoll_wait (Errno::EPERM)
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:383:in `run_once!'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:103:in `block in close'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:458:in `block in run_loop'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:455:in `handle_interrupt'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:455:in `run_loop'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/scheduler.rb:101:in `close'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/async/reactor.rb:27:in `scheduler_close'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/kernel/sync.rb:35:in `set_scheduler'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/kernel/sync.rb:35:in `ensure in Sync'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/async-2.21.0/lib/kernel/sync.rb:35:in `Sync'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/xyz-4.1.19/lib/xyz/runners/single_process.rb:51:in `start_actors'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/xyz-4.1.19/lib/xyz/runners/single_process.rb:28:in `run'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/xyz-4.1.19/lib/xyz/application.rb:96:in `start'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/xyz-4.1.19/lib/xyz/cli/cli.rb:23:in `start'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/thor-1.3.2/lib/thor/command.rb:28:in `run'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/thor-1.3.2/lib/thor/invocation.rb:127:in `invoke_command'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/thor-1.3.2/lib/thor.rb:538:in `dispatch'
        from /snap/xyz-protocol-simulator/x83/app/vendor/bundle/ruby/3.3.0/gems/thor-1.3.2/lib/thor/base.rb:584:in `start'
        from /snap/xyz-protocol-simulator/x83/app/bin/simulator:13:in `<top (required)>'
```


Running with `env -i strace --trace desc (which async-issue-360)` (Fish shell), this will give:

```strace
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d88d000
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=71519, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 71519, PROT_READ, MAP_PRIVATE, 3, 0) = 0x77b65d87b000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0 \0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 48, 848) = 48
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0I\17\357\204\3$\f\221\2039x\324\224\323\236S"..., 68, 896) = 68
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=2220400, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2264656, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x77b65d600000
mmap(0x77b65d628000, 1658880, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x28000) = 0x77b65d628000
mmap(0x77b65d7bd000, 360448, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1bd000) = 0x77b65d7bd000
mmap(0x77b65d816000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x215000) = 0x77b65d816000
mmap(0x77b65d81c000, 52816, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b65d81c000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d878000
openat(AT_FDCWD, "/sys/kernel/mm/transparent_hugepage/hpage_pmd_size", O_RDONLY) = 3
read(3, "2097152\n", 20)                = 8
close(3)                                = 0
mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d838000
mmap(NULL, 131072, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d5e0000
mmap(NULL, 1048576, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d4e0000
mmap(NULL, 8388608, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65cce0000
mmap(NULL, 67108864, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b658ce0000
mmap(NULL, 536870912, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b638ce0000
mmap(0xc000000000, 67108864, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xc000000000
mmap(NULL, 33554432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b636ce0000
mmap(NULL, 2165776, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b636acf000
mmap(0xc000000000, 4194304, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xc000000000
mmap(0x77b65d5e0000, 131072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b65d5e0000
mmap(0x77b65d560000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b65d560000
mmap(0x77b65d0e6000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b65d0e6000
mmap(0x77b65ad10000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b65ad10000
mmap(0x77b648e60000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x77b648e60000
mmap(NULL, 1048576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b6369cf000
mmap(NULL, 65536, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b65d87d000
mmap(NULL, 65536, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b6369bf000
mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x77b6361be000
mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x77b6359bd000
mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x77b6351bc000
mmap(NULL, 8392704, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x77b63417a000
fcntl(0, F_GETFL)                       = 0x402 (flags O_RDWR|O_APPEND)
mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b63413a000
fcntl(1, F_GETFL)                       = 0x8001 (flags O_WRONLY|O_LARGEFILE)
fcntl(2, F_GETFL)                       = 0x8001 (flags O_WRONLY|O_LARGEFILE)
openat(AT_FDCWD, "/etc/os-release", O_RDONLY|O_CLOEXEC) = 3
epoll_create1(EPOLL_CLOEXEC)            = 4
pipe2([5, 6], O_NONBLOCK|O_CLOEXEC)     = 0
epoll_ctl(4, EPOLL_CTL_ADD, 5, {events=EPOLLIN, data={u32=2709004344, u64=107175027941432}}) = 0
epoll_ctl(4, EPOLL_CTL_ADD, 3, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
read(3, "PRETTY_NAME=\"Ubuntu 22.04.5 LTS\""..., 4096) = 386
read(3, "", 3710)                       = 0
newfstatat(AT_FDCWD, "/proc/sys/fs/binfmt_misc/WSLInterop", 0xc00010cc68, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/run/WSL", 0xc00010cd38, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/meta/snap.yaml", 0xc00010ce08, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/snap/core/current/usr/share/locale-langpack/LC_MESSAGES/snappy.mo", 0xc00010d2e8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/snap/core/current/usr/share/locale/LC_MESSAGES/snappy.mo", 0xc00010d3b8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/share/locale-langpack/LC_MESSAGES/snappy.mo", 0xc00010d488, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/share/locale/LC_MESSAGES/snappy.mo", 0xc00010d558, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/snap/core/current/usr/share/locale-langpack/LC_MESSAGES/snappy.mo", 0xc00010d628, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/snap/core/current/usr/share/locale/LC_MESSAGES/snappy.mo", 0xc00010d6f8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/share/locale-langpack/LC_MESSAGES/snappy.mo", 0xc00010d7c8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/share/locale/LC_MESSAGES/snappy.mo", 0xc00010d898, 0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/etc/apparmor.d/cache", 0xc00010df18, 0) = -1 ENOENT (No such file or directory)
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
newfstatat(AT_FDCWD, "/usr/lib/snapd/apparmor_parser", 0xc0001e2038, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/sbin/apparmor_parser", 0xc0001e2108, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/bin/apparmor_parser", 0xc0001e21d8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/sbin/apparmor_parser", {st_mode=S_IFREG|0755, st_size=1548064, ...}, 0) = 0
newfstatat(AT_FDCWD, "/etc/apparmor.d/abi/4.0", 0xc0001e2378, AT_SYMLINK_NOFOLLOW) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/etc/apparmor.d/abi/3.0", {st_mode=S_IFREG|0644, st_size=1925, ...}, AT_SYMLINK_NOFOLLOW) = 0
openat(AT_FDCWD, "/proc/cgroups", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(7, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
read(7, "#subsys_name\thierarchy\tnum_cgrou"..., 4096) = 237
epoll_ctl(4, EPOLL_CTL_DEL, 7, 0xc00035f604) = 0
close(7)                                = 0
newfstatat(AT_FDCWD, "/usr/libexec/polkit-agent-helper-1", {st_mode=S_IFREG|S_ISUID|0755, st_size=18736, ...}, 0) = 0
mmap(0xc000400000, 4194304, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xc000400000
ioctl(1, TCGETS, 0xc0003ffbe4)          = -1 ENOTTY (Inappropriate ioctl for device)
ioctl(0, TCGETS, {B38400 opost isig icanon echo ...}) = 0
newfstatat(AT_FDCWD, "/var/lib/snapd/features/registries", 0xc0001e33b8, 0) = -1 ENOENT (No such file or directory)
mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77b6340fa000
newfstatat(AT_FDCWD, "/var/lib/snapd/features/registries", 0xc0001e3488, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/var/lib/snapd/features/registries", 0xc0001e3558, 0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/proc/cmdline", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(7, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
fstat(7, {st_mode=S_IFREG|0444, st_size=160, ...}) = 0
read(7, "BOOT_IMAGE=/boot/vmlinuz-6.5.0-4"..., 512) = 160
read(7, "", 352)                        = 0
epoll_ctl(4, EPOLL_CTL_DEL, 7, 0xc0003ffa4c) = 0
close(7)                                = 0
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
newfstatat(AT_FDCWD, "/snap/snapd/current/usr/bin/snap", {st_mode=S_IFREG|0755, st_size=17592392, ...}, 0) = 0
openat(AT_FDCWD, "/snap/snapd/current/usr/lib/snapd/info", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
read(7, "VERSION=2.66.1\nSNAPD_APPARMOR_RE"..., 4096) = 118
read(7, "", 3978)                       = 0
close(7)                                = 0
openat(AT_FDCWD, "/proc/sys/crypto/fips_enabled", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/snap/bin/async-issue-360", {st_mode=S_IFLNK|0777, st_size=13, ...}, AT_SYMLINK_NOFOLLOW) = 0
readlinkat(AT_FDCWD, "/snap/bin/async-issue-360", "/usr/bin/snap", 128) = 13
newfstatat(AT_FDCWD, "/var/lib/snapd/features/apparmor-prompting", 0xc0001e3898, 0) = -1 ENOENT (No such file or directory)
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
openat(AT_FDCWD, "/usr/lib/snapd/snapd", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
pread64(7, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0", 16, 0) = 16
pread64(7, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0ps\36\0\0\0\0\0"..., 64, 0) = 64
pread64(7, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 56, 64) = 56
pread64(7, "\3\0\0\0\4\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0"..., 56, 120) = 56
pread64(7, "\1\0\0\0\4\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 56, 176) = 56
pread64(7, "\1\0\0\0\5\0\0\0\0p\36\0\0\0\0\0\0p\36\0\0\0\0\0\0p\36\0\0\0\0\0"..., 56, 232) = 56
pread64(7, "\1\0\0\0\4\0\0\0\0000\300\0\0\0\0\0\0000\300\0\0\0\0\0\0000\300\0\0\0\0\0"..., 56, 288) = 56
pread64(7, "\1\0\0\0\6\0\0\0\360j\346\0\0\0\0\0\360z\346\0\0\0\0\0\360z\346\0\0\0\0\0"..., 56, 344) = 56
pread64(7, "\2\0\0\0\6\0\0\0000\34b\1\0\0\0\0000,b\1\0\0\0\0000,b\1\0\0\0\0"..., 56, 400) = 56
pread64(7, "\4\0\0\0\4\0\0\0p\3\0\0\0\0\0\0p\3\0\0\0\0\0\0p\3\0\0\0\0\0\0"..., 56, 456) = 56
pread64(7, "\4\0\0\0\4\0\0\0\220\3\0\0\0\0\0\0\220\3\0\0\0\0\0\0\220\3\0\0\0\0\0\0"..., 56, 512) = 56
pread64(7, "\7\0\0\0\4\0\0\0\360j\346\0\0\0\0\0\360z\346\0\0\0\0\0\360z\346\0\0\0\0\0"..., 56, 568) = 56
pread64(7, "S\345td\4\0\0\0p\3\0\0\0\0\0\0p\3\0\0\0\0\0\0p\3\0\0\0\0\0\0"..., 56, 624) = 56
pread64(7, "P\345td\4\0\0\0\250`\346\0\0\0\0\0\250`\346\0\0\0\0\0\250`\346\0\0\0\0\0"..., 56, 680) = 56
pread64(7, "Q\345td\6\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 56, 736) = 56
pread64(7, "R\345td\4\0\0\0\360j\346\0\0\0\0\0\360z\346\0\0\0\0\0\360z\346\0\0\0\0\0"..., 56, 792) = 56
pread64(7, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 64, 23575488) = 64
pread64(7, "\v\0\0\0\1\0\0\0\2\0\0\0\0\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0"..., 64, 23575552) = 64
pread64(7, "\23\0\0\0\7\0\0\0\2\0\0\0\0\0\0\0p\3\0\0\0\0\0\0p\3\0\0\0\0\0\0"..., 64, 23575616) = 64
pread64(7, "&\0\0\0\7\0\0\0\2\0\0\0\0\0\0\0\220\3\0\0\0\0\0\0\220\3\0\0\0\0\0\0"..., 64, 23575680) = 64
pread64(7, "9\0\0\0\7\0\0\0\2\0\0\0\0\0\0\0\264\3\0\0\0\0\0\0\264\3\0\0\0\0\0\0"..., 64, 23575744) = 64
pread64(7, "G\0\0\0\7\0\0\0\2\0\0\0\0\0\0\0\324\3\0\0\0\0\0\0\324\3\0\0\0\0\0\0"..., 64, 23575808) = 64
pread64(7, "X\0\0\0\366\377\377o\2\0\0\0\0\0\0\08\4\0\0\0\0\0\08\4\0\0\0\0\0\0"..., 64, 23575872) = 64
pread64(7, "b\0\0\0\v\0\0\0\2\0\0\0\0\0\0\0\0\6\0\0\0\0\0\0\0\6\0\0\0\0\0\0"..., 64, 23575936) = 64
pread64(7, "j\0\0\0\3\0\0\0\2\0\0\0\0\0\0\0\200\20\0\0\0\0\0\0\200\20\0\0\0\0\0\0"..., 64, 23576000) = 64
pread64(7, "r\0\0\0\377\377\377o\2\0\0\0\0\0\0\0\370\26\0\0\0\0\0\0\370\26\0\0\0\0\0\0"..., 64, 23576064) = 64
pread64(7, "\177\0\0\0\376\377\377o\2\0\0\0\0\0\0\0\330\27\0\0\0\0\0\0\330\27\0\0\0\0\0\0"..., 64, 23576128) = 64
pread64(7, "\216\0\0\0\4\0\0\0\2\0\0\0\0\0\0\0H\30\0\0\0\0\0\0H\30\0\0\0\0\0\0"..., 64, 23576192) = 64
pread64(7, "\230\0\0\0\4\0\0\0B\0\0\0\0\0\0\0\10g\36\0\0\0\0\0\10g\36\0\0\0\0\0"..., 64, 23576256) = 64
pread64(7, "\242\0\0\0\1\0\0\0\6\0\0\0\0\0\0\0\0p\36\0\0\0\0\0\0p\36\0\0\0\0\0"..., 64, 23576320) = 64
pread64(7, "\235\0\0\0\1\0\0\0\6\0\0\0\0\0\0\0 p\36\0\0\0\0\0 p\36\0\0\0\0\0"..., 64, 23576384) = 64
pread64(7, "\250\0\0\0\1\0\0\0\6\0\0\0\0\0\0\0Ps\36\0\0\0\0\0Ps\36\0\0\0\0\0"..., 64, 23576448) = 64
pread64(7, "\261\0\0\0\1\0\0\0\6\0\0\0\0\0\0\0`s\36\0\0\0\0\0`s\36\0\0\0\0\0"..., 64, 23576512) = 64
pread64(7, "\267\0\0\0\1\0\0\0\6\0\0\0\0\0\0\0\350%\300\0\0\0\0\0\350%\300\0\0\0\0\0"..., 64, 23576576) = 64
pread64(7, "\275\0\0\0\1\0\0\0\2\0\0\0\0\0\0\0\0000\300\0\0\0\0\0\0000\300\0\0\0\0\0"..., 64, 23576640) = 64
pread64(7, "\305\0\0\0\1\0\0\0\2\0\0\0\0\0\0\0\250`\346\0\0\0\0\0\250`\346\0\0\0\0\0"..., 64, 23576704) = 64
pread64(7, "\323\0\0\0\1\0\0\0\2\0\0\0\0\0\0\0(b\346\0\0\0\0\0(b\346\0\0\0\0\0"..., 64, 23576768) = 64
pread64(7, "\335\0\0\0\10\0\0\0\3\4\0\0\0\0\0\0\360z\346\0\0\0\0\0\360j\346\0\0\0\0\0"..., 64, 23576832) = 64
pread64(7, "\343\0\0\0\16\0\0\0\3\0\0\0\0\0\0\0\360z\346\0\0\0\0\0\360j\346\0\0\0\0\0"..., 64, 23576896) = 64
pread64(7, "\357\0\0\0\17\0\0\0\3\0\0\0\0\0\0\0\370z\346\0\0\0\0\0\370j\346\0\0\0\0\0"..., 64, 23576960) = 64
pread64(7, "\373\0\0\0\1\0\0\0\3\0\0\0\0\0\0\0\0{\346\0\0\0\0\0\0k\346\0\0\0\0\0"..., 64, 23577024) = 64
pread64(7, "\10\1\0\0\6\0\0\0\3\0\0\0\0\0\0\0000,b\1\0\0\0\0000\34b\1\0\0\0\0"..., 64, 23577088) = 64
pread64(7, "\254\0\0\0\1\0\0\0\3\0\0\0\0\0\0\0 .b\1\0\0\0\0 \36b\1\0\0\0\0"..., 64, 23577152) = 64
pread64(7, "\21\1\0\0\1\0\0\0\3\0\0\0\0\0\0\0\0000b\1\0\0\0\0\0 b\1\0\0\0\0"..., 64, 23577216) = 64
pread64(7, "\27\1\0\0\1\0\0\0\3\0\0\0\0\0\0\0\320\236c\1\0\0\0\0\320\216c\1\0\0\0\0"..., 64, 23577280) = 64
pread64(7, "%\1\0\0\1\0\0\0\3\0\0\0\0\0\0\0\340\247c\1\0\0\0\0\340\227c\1\0\0\0\0"..., 64, 23577344) = 64
pread64(7, "0\1\0\0\10\0\0\0\3\0\0\0\0\0\0\0@\312g\1\0\0\0\08\272g\1\0\0\0\0"..., 64, 23577408) = 64
pread64(7, "5\1\0\0\10\0\0\0\3\0\0\0\0\0\0\0\340\36k\1\0\0\0\08\272g\1\0\0\0\0"..., 64, 23577472) = 64
pread64(7, "?\1\0\0\1\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\08\272g\1\0\0\0\0"..., 64, 23577536) = 64
pread64(7, "\1\0\0\0\3\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0l\272g\1\0\0\0\0"..., 64, 23577600) = 64
pread64(7, "\0.shstrtab\0.interp\0.note.gnu.pro"..., 334, 23575148) = 334
pread64(7, "\4\0\0\0\20\0\0\0\5\0\0\0", 12, 880) = 12
pread64(7, "\4\0\0\0\24\0\0\0\3\0\0\0", 12, 912) = 12
pread64(7, "GNU\0", 4, 924)             = 4
pread64(7, "\375\211|\237\371\351\220\225\250Y\234a\257\373SG\0316\253A", 20, 928) = 20
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 18 entries */, 8192) = 528
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/caps", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/dbus", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/domain", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 12 entries */, 8192) = 424
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/domain/attach_conditions", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 80
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/file", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/io_uring", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/ipc", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 80
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/mount", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 4 entries */, 8192) = 104
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/namespaces", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 5 entries */, 8192) = 136
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/network", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 4 entries */, 8192) = 112
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/network_v8", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 80
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/policy", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 8 entries */, 8192) = 264
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/policy/unconfined_restrictions", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 5 entries */, 8192) = 152
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/policy/versions", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 7 entries */, 8192) = 168
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/ptrace", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/query", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 80
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/query/label", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 5 entries */, 8192) = 144
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/rlimit", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/signal", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
getdents64(7, 0xc000434000 /* 3 entries */, 8192) = 72
getdents64(7, 0xc000434000 /* 0 entries */, 8192) = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/policy/permstable32", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
fstat(7, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
read(7, "allow deny subtree cond kill com"..., 512) = 79
read(7, "", 433)                        = 0
close(7)                                = 0
openat(AT_FDCWD, "/sys/kernel/security/apparmor/features/policy/notify/user", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
newfstatat(AT_FDCWD, "/usr/lib/snapd/apparmor_parser", 0xc0001e3e48, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/sbin/apparmor_parser", 0xc0001e3f18, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/bin/apparmor_parser", 0xc000438038, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/sbin/apparmor_parser", {st_mode=S_IFREG|0755, st_size=1548064, ...}, 0) = 0
newfstatat(AT_FDCWD, "/etc/apparmor.d/abi/4.0", 0xc0004381d8, AT_SYMLINK_NOFOLLOW) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/etc/apparmor.d/abi/3.0", {st_mode=S_IFREG|0644, st_size=1925, ...}, AT_SYMLINK_NOFOLLOW) = 0
newfstatat(AT_FDCWD, "/usr/sbin/apparmor_parser", {st_mode=S_IFREG|0755, st_size=1548064, ...}, 0) = 0
openat(AT_FDCWD, "/proc/self/mountinfo", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(7, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
read(7, "23 29 0:21 / /sys rw,nosuid,node"..., 4096) = 4091
read(7, "163 29 7:10 / /snap/core22/1663 "..., 4096) = 4008
read(7, "190 29 7:20 / /snap/firefox/5437"..., 4096) = 834
read(7, "", 3262)                       = 0
epoll_ctl(4, EPOLL_CTL_DEL, 7, 0xc0003ff85c) = 0
close(7)                                = 0
openat(AT_FDCWD, "/etc/fstab", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
read(7, "# /etc/fstab: static file system"..., 4096) = 473
read(7, "", 3623)                       = 0
close(7)                                = 0
openat(AT_FDCWD, "/proc/self/mountinfo", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(7, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
read(7, "23 29 0:21 / /sys rw,nosuid,node"..., 4096) = 4091
read(7, "163 29 7:10 / /snap/core22/1663 "..., 4096) = 4008
read(7, "190 29 7:20 / /snap/firefox/5437"..., 4096) = 834
read(7, "", 3262)                       = 0
epoll_ctl(4, EPOLL_CTL_DEL, 7, 0xc0003ff8d4) = 0
close(7)                                = 0
openat(AT_FDCWD, "/proc/sys/kernel/seccomp/actions_avail", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(7, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
fstat(7, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
read(7, "kill_process kill_thread trap er"..., 512) = 63
read(7, "", 449)                        = 0
epoll_ctl(4, EPOLL_CTL_DEL, 7, 0xc0003ff804) = 0
close(7)                                = 0
openat(AT_FDCWD, "/dev/null", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
pipe2([8, 9], O_CLOEXEC)                = 0
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(8, F_GETFL)                       = 0 (flags O_RDONLY)
fcntl(8, F_SETFL, O_RDONLY|O_NONBLOCK)  = 0
epoll_ctl(4, EPOLL_CTL_ADD, 9, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727624, u64=131624441480840}}) = 0
fcntl(9, F_GETFL)                       = 0x1 (flags O_WRONLY)
fcntl(9, F_SETFL, O_WRONLY|O_NONBLOCK)  = 0
pipe2([10, 11], O_CLOEXEC)              = 0
epoll_ctl(4, EPOLL_CTL_ADD, 10, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727384, u64=131624441480600}}) = 0
fcntl(10, F_GETFL)                      = 0 (flags O_RDONLY)
fcntl(10, F_SETFL, O_RDONLY|O_NONBLOCK) = 0
epoll_ctl(4, EPOLL_CTL_ADD, 11, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727144, u64=131624441480360}}) = 0
fcntl(11, F_GETFL)                      = 0x1 (flags O_WRONLY)
fcntl(11, F_SETFL, O_WRONLY|O_NONBLOCK) = 0
fcntl(9, F_GETFL)                       = 0x801 (flags O_WRONLY|O_NONBLOCK)
fcntl(9, F_SETFL, O_WRONLY)             = 0
fcntl(11, F_GETFL)                      = 0x801 (flags O_WRONLY|O_NONBLOCK)
fcntl(11, F_SETFL, O_WRONLY)            = 0
pipe2([12, 13], O_CLOEXEC)              = 0
close(13)                               = 0
read(12, "", 8)                         = 0
close(12)                               = 0
close(7)                                = 0
epoll_ctl(4, EPOLL_CTL_DEL, 9, 0xc0003ff6cc) = 0
close(9)                                = 0
epoll_ctl(4, EPOLL_CTL_DEL, 11, 0xc0003ff6cc) = 0
close(11)                               = 0
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=2957445, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---
openat(AT_FDCWD, "/var/lib/snapd/system-key", O_RDONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
fstat(7, {st_mode=S_IFREG|0644, st_size=1173, ...}) = 0
read(7, "{\"version\":11,\"build-id\":\"fd897c"..., 1174) = 1173
read(7, "", 1)                          = 0
close(7)                                = 0
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
openat(AT_FDCWD, "/var/lib/snapd/inhibit/async-issue-360.lock", O_RDONLY|O_NOFOLLOW|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
flock(7, LOCK_SH)                       = 0
read(7, "", 512)                        = 0
readlinkat(AT_FDCWD, "/snap/async-issue-360/current", "x15", 128) = 3
openat(AT_FDCWD, "/snap/async-issue-360/x15/meta/snap.yaml", O_RDONLY|O_CLOEXEC) = 8
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
fstat(8, {st_mode=S_IFREG|0644, st_size=709, ...}) = 0
read(8, "name: async-issue-360\nversion: 0"..., 710) = 709
read(8, "", 1)                          = 0
close(8)                                = 0
newfstatat(AT_FDCWD, "/snap/async-issue-360/x15/meta/hooks", 0xc0004393b8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/var/lib/snapd/snaps/async-issue-360_x15.snap", {st_mode=S_IFREG|0600, st_size=36933632, ...}, AT_SYMLINK_NOFOLLOW) = 0
readlinkat(AT_FDCWD, "/proc/self/exe", "/usr/bin/snap", 128) = 13
newfstatat(AT_FDCWD, "/usr/lib/snapd/snap-confine", {st_mode=S_IFREG|S_ISUID|0755, st_size=150824, ...}, 0) = 0
openat(AT_FDCWD, "/var/lib/snapd/sequence/async-issue-360.json", O_RDONLY|O_CLOEXEC) = 8
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
fstat(8, {st_mode=S_IFREG|0644, st_size=255, ...}) = 0
read(8, "{\"sequence\":[{\"name\":\"async-issu"..., 512) = 255
read(8, "", 257)                        = 0
close(8)                                = 0
close(8)                                = 0
close(8)                                = 0
newfstatat(AT_FDCWD, "/etc/nsswitch.conf", {st_mode=S_IFREG|0644, st_size=582, ...}, 0) = 0
newfstatat(AT_FDCWD, "/", {st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
openat(AT_FDCWD, "/etc/nsswitch.conf", O_RDONLY|O_CLOEXEC) = 8
newfstatat(8, "", {st_mode=S_IFREG|0644, st_size=582, ...}, AT_EMPTY_PATH) = 0
read(8, "# /etc/nsswitch.conf\n#\n# Example"..., 4096) = 582
read(8, "", 4096)                       = 0
newfstatat(8, "", {st_mode=S_IFREG|0644, st_size=582, ...}, AT_EMPTY_PATH) = 0
close(8)                                = 0
openat(AT_FDCWD, "/etc/passwd", O_RDONLY|O_CLOEXEC) = 8
newfstatat(8, "", {st_mode=S_IFREG|0644, st_size=3231, ...}, AT_EMPTY_PATH) = 0
lseek(8, 0, SEEK_SET)                   = 0
read(8, "root:x:0:0:root:/root:/usr/bin/f"..., 4096) = 3231
close(8)                                = 0
newfstatat(AT_FDCWD, "/home/user/snap", {st_mode=S_IFDIR|0700, st_size=4096, ...}, 0) = 0
newfstatat(AT_FDCWD, "/home/user/snap/async-issue-360/x15", {st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
newfstatat(AT_FDCWD, "/home/user/snap/async-issue-360/common", {st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
readlinkat(AT_FDCWD, "/home/user/snap/async-issue-360/current", "x15", 128) = 3
openat(AT_FDCWD, "/proc/self/mountinfo", O_RDONLY|O_CLOEXEC) = 8
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
fcntl(8, F_GETFL)                       = 0x8000 (flags O_RDONLY|O_LARGEFILE)
fcntl(8, F_SETFL, O_RDONLY|O_NONBLOCK|O_LARGEFILE) = 0
read(8, "23 29 0:21 / /sys rw,nosuid,node"..., 4096) = 4091
read(8, "163 29 7:10 / /snap/core22/1663 "..., 4096) = 4008
read(8, "190 29 7:20 / /snap/firefox/5437"..., 4096) = 834
read(8, "", 3262)                       = 0
epoll_ctl(4, EPOLL_CTL_DEL, 8, 0xc0003ff564) = 0
close(8)                                = 0
newfstatat(AT_FDCWD, "/run/user/1000", {st_mode=S_IFDIR|0700, st_size=520, ...}, 0) = 0
newfstatat(AT_FDCWD, "/var/lib/snapd/save/snap/async-issue-360", 0xc000439bd8, 0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/proc/2957386/cgroup", O_RDONLY|O_CLOEXEC) = 8
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = -1 EPERM (Operation not permitted)
read(8, "13:devices:/user.slice\n12:perf_e"..., 4096) = 371
close(8)                                = 0
newfstatat(AT_FDCWD, "/run/user/1000/dbus-session", 0xc000439ca8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/run/user/1000/bus", {st_mode=S_IFSOCK|0666, st_size=0, ...}, 0) = 0
newfstatat(AT_FDCWD, "/run/user/1000/bus", {st_mode=S_IFSOCK|0666, st_size=0, ...}, 0) = 0
epoll_ctl(4, EPOLL_CTL_ADD, 8, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727864, u64=131624441481080}}) = 0
write(8, "AUTH\r\n", 6)                 = 6
read(8, 0xc0004cb000, 4096)             = -1 EAGAIN (Resource temporarily unavailable)
epoll_pwait(4, [], 128, 0, NULL, 0)     = 0
epoll_pwait(4, [{events=EPOLLIN, data={u32=2709004344, u64=107175027941432}}], 128, -1, NULL, 0) = 1
read(5, "\0", 16)                       = 1
epoll_pwait(4, [], 128, 0, NULL, 0)     = 0
epoll_pwait(4, [], 128, 1, NULL, 702606225321246) = 0
openat(AT_FDCWD, "/proc/2957386/cgroup", O_RDONLY|O_CLOEXEC) = 9
epoll_ctl(4, EPOLL_CTL_ADD, 9, {events=EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, data={u32=873727384, u64=131624441480600}}) = -1 EPERM (Operation not permitted)
read(9, "13:devices:/user.slice\n12:perf_e"..., 4096) = 559
close(9)                                = 0
close(7)                                = 0
fcntl(0, F_GETFD)                       = 0
fcntl(1, F_GETFD)                       = 0
fcntl(2, F_GETFD)                       = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7b9535301000
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=71519, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 71519, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7b95352ef000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libudev.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\0\0\0\0\0\0\0\0"..., 832) = 832
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=166240, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 170272, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7b95352c5000
mmap(0x7b95352c9000, 106496, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x4000) = 0x7b95352c9000
mmap(0x7b95352e3000, 36864, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e000) = 0x7b95352e3000
mmap(0x7b95352ed000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x27000) = 0x7b95352ed000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0 \0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 48, 848) = 48
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0I\17\357\204\3$\f\221\2039x\324\224\323\236S"..., 68, 896) = 68
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=2220400, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2264656, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7b9535000000
mmap(0x7b9535028000, 1658880, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x28000) = 0x7b9535028000
mmap(0x7b95351bd000, 360448, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1bd000) = 0x7b95351bd000
mmap(0x7b9535216000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x215000) = 0x7b9535216000
mmap(0x7b953521c000, 52816, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7b953521c000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7b95352c2000
newfstatat(AT_FDCWD, "/proc/1/root/snap", 0x7ffff1daf480, AT_SYMLINK_NOFOLLOW) = -1 EACCES (Permission denied)
write(2, "cannot fstatat canonical snap di"..., 59cannot fstatat canonical snap directory: Permission denied
) = 59
+++ exited with 1 +++
```
