# Vagrant 1.5 で Windows の対応状況がよくなってた

Windows + Vagrant で synced_folder する必要ができて難儀だなと思ってたら、 Vagrant 1.5 でそこそこ解決してた。

Vagrant(VirtualBox provider) で synced_folder 指定して type オプションを明示しない場合、VirtualBox 標準の shared folders が使われる。ただし shared folders はゲスト OS からの read write ともに相当遅く、「ゲスト OS で httpd を立ち上げて Windows の使い慣れたエディタで編集しながら見え方を確かめたい」という用途には適していると言えない。

ホスト OS が Mac だったら [NFS type](https://docs.vagrantup.com/v2/synced-folders/nfs.html) を指定できるので、そこそこの速度を期待できたが、残念ながら Windows 環境では使えない。

## SMB synced_folder

どうしたものかなと思っていたら、先日リリースの Vagrant 1.5 で [Windows 対応状況の強化がされていた](http://www.vagrantup.com/blog/feature-preview-vagrant-1-5-hyperv.html)。 synced_folder に SMB type が指定できるようになり、速度を期待できるようになった。

使い方は他の synced_folder と同じように記述して、 SMB type を明示的に指定する。

```
config.vm.synced_folder ".", "/vagrant", type: "smb"
```

省略した場合でも VM の shared folder 機能が存在しない場合には、 Windows では SMB が使用される。 OS を問わず通用する Vagrantfile を書きたいなら type を指定しないことが推奨されているが、環境の選択肢がある程度わかっていて速度を期待したい場合、雑に判定を入れてもよさそう。

```
if RUBY_PLATFORM =~ /win32/
  config.vm.synced_folder ".", "/vagrant", type: "smb"
else
  config.vm.synced_folder ".", "/vagrant", type: "nfs"
end
```

## RSync synced_folder

Windows の synced_folder を解決するのに調べてて最初に辿り着いたのが、実は RSync type だった。これも同じく [Vagrant 1.5 での新機能](https://www.vagrantup.com/blog/feature-preview-vagrant-1-5-rsync.html)。こちらも使い方は type を rsync にする。

```
config.vm.synced_folder ".", "/vagrant", type: "rsync"
```

ファイルシステム自体はゲスト OS 上にあるのでパフォーマンスの劣化もないし、 guest additions がインストールできないゲスト OS に対してもファイルを転送することができる。 vagrant rsync コマンドで都度ファイルの同期をとることもできるし、 vagrant rsync-auto コマンドでファイルの変更を自動的に検出し、転送も可能になる。

rsync-auto での自動検出は遅いと [GitHub 上でも話題になってて](https://github.com/mitchellh/vagrant/issues/3249)、より高速な自動 RSync 起動プラグイン [vagrant-gatling-rsync](https://github.com/smerrill/vagrant-gatling-rsync) が作られている。近々コアにもマージされるようなので、 synced_folder に RSync を使うならもう少し待った方がいいかも。

ただ、 Windows に RSync を入れようとすると Cygwin からインストールなりになって非常に面倒なので、結局 SMB type 使おうってことに。
