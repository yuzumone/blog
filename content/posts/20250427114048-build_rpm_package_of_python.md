+++
title = "Build RPM Package of Python"
author = ["yuzumone"]
date = 2020-07-05
slug = "build-rpm-package-of-python"
tags = ["Python"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1577702312706-e23ff063064f?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8OHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Brandable Box on Unsplash" src="https://images.unsplash.com/photo-1577702312706-e23ff063064f?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8OHx8fGVufDB8fHx8fA%3D%3D" >}}

Python パッケージの RPM 化といえば bdist_rpm がありますが，この場合依存パッケージも PRM 化する必要があります． <br/>
ということで venv でいい感じにパッケージ化するときの spec ファイルの内容をメモ． <br/>

ということで下記です． <br/>

```spec
%define venv_name package_name
%define venv_install_dir /opt/%{venv_name}
%define venv_dir %{buildroot}%{venv_install_dir}
%define venv_bin %{venv_dir}/bin
%define __prelink_undo_cmd %{nil}
%undefine __arch_install_post
%global __os_install_post %(echo '%{__os_install_post}' | sed -e 's!/usr/lib[^[:space:]]*/brp-python-bytecompile[[:space:]].*$!!g')
Name:       package_name
Version:    %{package_version}
Release:    1%{?dist}
Summary:    package_summary
URL:        https://github.com/hoge/huga
Source0:    ./
Requires:   python3%description
long_package_desccription%prep
rm -rf %{buildroot}/*%install
install -d %{buildroot}%{venv_install_dir} %{buildroot}/usr/sbin
python3 -m venv %{venv_dir}
%{venv_bin}/python %{venv_bin}/pip install --upgrade pip
%{venv_bin}/python %{venv_bin}/pip install venvctrl
cd %{_topdir}
%{venv_bin}/python setup.py install
ln -sf %{venv_install_dir}/bin/package_name %{buildroot}/usr/sbin/package_name
find %{buildroot} -name "RECORD" -exec rm -rf {} \;
%{venv_bin}/venvctrl-relocate --source=%{venv_dir} --destination=%{venv_install_dir}
find %{venv_dir}/lib -type f -name "*.so" | xargs -r strip%files
%{venv_install_dir}
/usr/sbin/package_name%clean
rm -rf %{buildroot}
```

軽く解説を書くと /opt 配下に pip install をやったあと /usr/sbin 配下にシンボリックリンクを張っています． <br/>
CUI ツールなどでよくあるパターンかと思います． <br/>
Cent6 と Cent7 で動くことは確認しているので大丈夫のはず． <br/>

あと当たり前なんですけど venv が activate 状態で rpmbuild すると venv の Python でパッケージが作成されるので注意してください． <br/>

