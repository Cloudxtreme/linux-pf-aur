# Maintainer: Thaodan <theodorstormgrade@gmail.com>
# Submitter: Christos Nouskas <nous at archlinux dot us>
# PKGBUILD assembled from kernel26
# Some lines from  kernel26-bfs and kernel26-ck
# Credits to respective maintainers
_major=4
_minor=1
#_patchlevel=0
#_subversion=1
_basekernel=${_major}.${_minor}
_srcname=linux-${_major}.${_minor}
pkgbase=linux-pf
_pfrel=4
_kernelname=-pf
_pfpatchhome="http://pf.natalenko.name/sources/${_basekernel}/"
_pfpatchname="patch-${_basekernel}${_kernelname}${_pfrel}"
_aufs3="https://github.com/sfjro/aufs4-standalone"
_CPUSUFFIXES_KBUILD=(
  CORE2 K7 K8 K10 BARCELONA BOBCAT BULLDOZER PILEDRIVER PSC
  ATOM PENTIUMII PENTIUMIII PENTIUMM PENTIUM4 NEHALEM SANDYBRIDGE
  IVYBRIDGE HASWELL BROADWELL SILVERMONT)
_CPUSUFFIXES=( core2 k7 k8 k10 barcelona bobcat
	       bulldozer piledriver psc atom p2 p3 pm p4
	       nehalem sandybridge ivybridge haswell broadwell silvermont)

### PATCH AND BUILD OPTIONS
#
# taken from graysky linux-ck see: https://aur.archlinux.org/packages/linux-ck
# Set these variables to ANYTHING (yes or no or 1 or 0 or "I like icecream") to enable them
#
_NUMA_off=yes		# Disable NUMA in kernel config

# batch mode:
# enable batch mode to stop the pkgbuild from asking you what to do and just use defaults
_BATCH_MODE=n
# if $srcdir/batch_opts is found enable $BATCH_MODE and set MARCH to M$CPU


### DOCS
# Starting with the 3.6.6-3 release, this package ships with the kernel-3x-gcc47-x.patch.
# This allows users an expanded scope of CPU specific options.
# Consult the following resources to understand which option is right for you application:
#
# http://en.gentoo-wiki.com/wiki/Safe_Cflags/Intel
# http://en.gentoo-wiki.com/wiki/Safe_Cflags/AMD
# http://www.linuxforge.net/docs/linux/linux-gcc.php 
# http://gcc.gnu.org/onlinedocs/gcc/i386-and-x86_002d64-Options.html



# DETAILS FOR using 'make localmodconfig'
# As of mainline 2.6.32, running with this option will only build the modules that you currently have
# probed in your system VASTLY reducing the number of modules built and the build time to do it.
#
# WARNING - make CERTAIN that all modules are modprobed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware, give my module_db script
# a try: http://aur.archlinux.org/packages.php?ID=41689  Note that if you use my script, this PKGBUILD 
# will auto run the 'sudo modprobed_db reload' for you to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed_db

# DETAILS FOR running kernel's config
# Enabling this option will use the .config of the RUNNING kernel rather than the ARCH defaults.
# Useful when the package gets updated and you already went through the trouble of customizing your
# config options.  NOT recommended when a new kernel is released, but again, convenient for package bumps.

# DRTAILS FOR _NUMA_off=yes
# Since 99.9% of users do not have multiple CPUs but do have multiple cores in one CPU
# see, https://bugs.archlinux.org/task/31187

pkgname=('linux-pf')
true && pkgname=('linux-pf' 'linux-pf-headers')
pkgver=${_basekernel}.${_pfrel}
pkgrel=2
arch=('i686' 'x86_64')
url="http://pf.natalenko.name/"
license=('GPL2')
options=('!strip')
makedepends=('git' 'xmlto' 'docbook-xsl' 'xz' 'bc' 'kmod')
source=("ftp://www.kernel.org/pub/linux/kernel/v${_major}.x/linux-${_basekernel}.tar.xz"
	'config' 'config.x86_64'		# the main kernel config files
	'linux.preset'			        # standard config files for mkinitcpio ramdisk
	'change-default-console-loglevel.patch'
	"${_pfpatchhome}${_pfpatchname}.xz"	# the -pf patchset
        "git+$_aufs3#branch=aufs4.1"
        'bfs_gc_remove_resched_closest_idle.patch'
       )


prepare() {
  cd "${srcdir}/linux-${_basekernel}"
  msg "Applying pf-kernel patch"
  patch -Np1 < ${srcdir}/${_pfpatchname}
  msg "applying aufs3 patches"
  cd "$srcdir/${_aufs3##*/}"
#  git checkout origin/aufs${_basekernel} || _aufs3checkout=KRAKRA
  if [[ ${_aufs3checkout} = "KRAKRA" ]]; then
      echo
      msg "AUFS3 not yet ported to version ${_basekernel}!"
      msg "Skipping related patches."
      echo
      cd ..
  else
    #        mkdir -p "${pkgdir}/usr/lib/modules/build/${_kernver}/include"
    #        mv include/linux/Kbuild "${pkgdir}/usr/lib/modules/build/${_kernver}/include/"
    rm include/uapi/linux/Kbuild
    cp -a {Documentation,fs,include} ${srcdir}/linux-${_basekernel}/
    cd ../linux-${_basekernel}
    msg "Patching aufs3"
    for _patch in ${srcdir}/${_aufs3##*/}/*.patch; do
      patch -Np1 -i ${_patch} || _aufs3fail=KRAKRA
    done
    if [[ ${_aufs3fail} = "KRAKRA" ]]; then
	echo
    	warning "Not all aufs3 patches applied correctly. Ignore this if you won't use AUFS."
    	warning "Otherwise, press CTRL-C now and fix manually"
	warning "Remind me to enable it again if it's aviable again"
    	echo
    fi
  fi
  
  # Interactive governor patch
  # broken for now
  #   msg "Interactice CPU governor patch"
  #   mv ${srcdir}/gist*/gistfile1.diff ${srcdir}
  #   patch -Np1 < ${srcdir}/gistfile1.diff

  # linux-ARCH patches

  # add latest fixes from stable queue, if needed
  # http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git
  

  # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
  # remove this when a Kconfig knob is made available by upstream
  # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
  patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"

  # end linux-ARCH patches


  # added gcc 4.7.1 support for Kconfig and menuconfig
  # now inclued in pf patchset

  # fix bfs issue
  patch -Np1 -i "${srcdir}/bfs_gc_remove_resched_closest_idle.patch"
  
  if [ "$CARCH" = "x86_64" ]; then
	cat "${startdir}/config.x86_64" >| .config
  else
	cat "${startdir}/config" >| .config
  fi

  sed -ri "s|SUBLEVEL = 0|SUBLEVEL = $_pfrel|" Makefile


  # Set EXTRAVERSION to -pf
  sed -ri "s|^(EXTRAVERSION =).*|\1|" Makefile
  _arch=$CARCH

  
  # disable NUMA since 99.9% of users do not have multiple CPUs but do have multiple cores in one CPU
  # see, https://bugs.archlinux.org/task/31187
  if [ -n "$_NUMA_off" ] && [ "${CARCH}" = "x86_64" ]; then
     sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
      -i -e '/CONFIG_AMD_NUMA=y/d' \
      -i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
      -i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
      -i -e '/# CONFIG_NUMA_EMU is not set/d' \
      -i -e '/CONFIG_NODES_SHIFT=6/d' \
      -i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
      -i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
      -i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
  fi

  # Don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # If the following is set, stop right there. We only need the headers for
  # dependent drivers' compiling (nvidia, virtualbox etc)

  # get kernel version
  #make prepare
}

build() {
  cd "${srcdir}/linux-${_basekernel}"

  # enable $_BATCH_MODE if batch_opts is found in $srcdir
  if [[ -e $srcdir/batch_opts ]] ; then
      source "$srcdir/batch_opts"
      if [[ "$CPU" ]] ; then # enable cpu optimisations acording to $CPU and enable pkgopt
          sed -e "s|# CONFIG_M$CPU is not set|CONFIG_M$CPU=y|" \
              -e '/CONFIG_GENERIC_CPU=y/d' \
              -i "$srcdir/linux-${_basekernel}/.config"
          export _PKGOPT=y
      fi

      _BATCH_MODE=y
  fi
  
  #----------------------------------------
  if [[ "$_BATCH_MODE" != "y" ]]; then		# for batch building
      echo
      echo "======================================================="
      msg "You might be prompted below for some config options"
      echo "======================================================="
      echo
      msg "Hit <Y> to use your running kernel's config"
      echo "    (needs IKCONFIG and IKCONFIG_PROC)"
      msg "Hit <L> to run 'make localmodconfig'"
      msg "Hit <N> (or just <ENTER>) to build an all-inclusive kernel like stock -ARCH"
      echo "    (warning: it can take a looong time)"
      echo
      read answer
      shopt -s nocasematch
      if [[ "$answer" = "y" ]]; then
          if [[ -s /proc/config.gz ]]; then
	      msg "Extracting config from /proc/config.gz..."
	      zcat /proc/config.gz >| ./.config
          else
	    msg "running 'sudo modprobe configs'"
	    sudo modprobe configs
            if [[ -s /proc/config.gz ]]; then
	        msg "Extracting config from /proc/config.gz..."
	        zcat /proc/config.gz >| ./.config
	    else
	      msg "You kernel was not compiled with IKCONFIG_PROC."
	      msg "Attempting to run /usr/bin/modprobed_db recall from modprobe_db..."
	      if [ -e /usr/bin/modprobed_db ]; then
	          sudo /usr/bin/modprobed_db recall
	      else
	        msg "modprobed_db not installed, running make localmodconfig instead..."
	        make localmodconfig
	      fi
            fi
          fi
      elif [[ "$answer" = "l" ]]; then
          # Copied from kernel26-ck's PKGBUILD
          msg "Attempting to run /usr/bin/reload_database with sudo from modprobe_db..."
          if [ -e /usr/bin/modprobed_db ]; then
	      sudo /usr/bin/modprobed_db recall
          fi
          msg "Running 'make localmodconfig'..."
          make localmodconfig
      else
        msg "Using stock ARCH kernel .config (with BFS, BFQ and TuxOnIce enabled)."
      fi
      
      # Make some good use of MAKEFLAGS
      # MAKEFLAGS=`grep -v '#' /etc/makepkg.conf | grep MAKEFLAGS= | sed s/MAKEFLAGS=// | sed s/\"//g`
      
      # make prepare
      
      # Options for additional configuration
      echo
      msg "Kernel configuration options before build:"
      echo "    <M> make menuconfig (console menu)"
      echo "    <N> make nconfig (newer alternative to menuconfig)"
      echo "    <G> make gconfig (needs gtk)"
      echo "    <X> make xconfig (needs qt)"
      echo "    <O> make oldconfig"
      echo "    <L> make localyesconfig"
      echo "    <ENTER> to skip configuration and use stock -ARCH defaults"
      read answer
      case "$answer" in
        m) make menuconfig
	   ;;
        g) make gconfig
	   ;;
        x) make xconfig
	   ;;
        n) make nconfig
	   ;;
        o) make oldconfig
           ;;
        l) make localyesconfig
	   ;;
        default)
	   ;;
      esac
      cp -v .config ${startdir}/config.local
      for _cpusuffix_kbuild in ${_CPUSUFFIXES_KBUILD[@]} ; do
	_egrepstring="${_egrepstring}M${_cpusuffix_kbuild}=y|"
      done
      CPU=$(egrep "${_egrepstring}CONFIG_GENERIC_CPU=y|M686=y|CONFIG_MNATIVE=y" ./.config)
      CPU=$(sed -e "s/CONFIG_M\(.*\)=y/\1/" <<<$CPU)
      CPU=$(sed -e "s/CONFIG_GENERIC_CPU=y/GENERIC/" <<<$CPU)
      CPU=$(sed -e "s/^686$/GENERIC/" <<<$CPU)
      cp -f .config ${startdir}/config.$CPU-$CARCH
      
      # Give option to rename package according to CPU
      echo
      if [[ "$CPU" != "GENERIC" ]]; then
          lcpu=$(tr '[:upper:]' '[:lower:]' <<< $CPU)
          lcpu=$(sed -e "s/entium//" <<<$lcpu)
          echo "=============================================================="
          msg "An non-generic CPU was selected for this kernel."
          echo
          msg "Hit <G>     :  to create a generic package named linux-pf"
          msg "Hit <ENTER> :  to create a package named after the selected CPU"
          msg "               (linux-pf-${lcpu} - recommended default)"
          echo
          msg "This option affects ONLY the package name. Whether or not the"
          msg "kernel is optimized was determined at the previous config step."
          msg "Also note that CPUs newer than CORE2 or K8 will be replaced by"
          msg "by core2 or k8 respectively in the package name."
          echo "=============================================================="
          read answer
          shopt -s nocasematch
          if [[ "$answer" != "g" ]]; then
	      export _PKGOPT=y
          fi
      fi
      
  fi  # batch check ends here
  export CPU
  #----------------------------------------

  # Strip config of uneeded localversion
  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
    sed -i "s/CONFIG_LOCALVERSION_AUTO=.*/CONFIG_LOCALVERSION_AUTO=n/" ./.config
  fi

  # rewrite configuration
  yes "" | make config >/dev/null

  # Build
  # Want extreme and non-sensical optimization? Uncomment the following line!
  # export KCFLAGS="-march=native -Ofast"
  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

package_linux-pf() {
 _pkgdesc="Linux kernel and modules with the pf-kernel patch [-ck patchset (BFS included), TuxOnIce, BFQ] and aufs3."
 pkgdesc=${_pkgdesc}
 groups=('base')
 backup=(etc/mkinitcpio.d/${pkgbase}.preset)
 depends=('coreutils' 'linux-firmware' 'kmod>=9-2' 'mkinitcpio>=0.7')
 optdepends=('linux-docs: Kernel hackers manual - HTML documentation that comes with the Linux kernel.'
	    'crda: to set the correct wireless channels of your country'
	    'pm-utils: utilities and scripts for suspend and hibernate power management'
	    'tuxonice-userui: TuxOnIce userspace user interface'
	    'hibernate-script: set of scripts for managing TuxOnIce, hibernation and suspend to RAM'
	    'nvidia-pf: NVIDIA drivers for linux-pf'
	    'nvidia-beta-all: NVIDIA drivers for all installed kernels'
	    'modprobed_db: Keeps track of EVERY kernel module that has ever been probed. Useful for make localmodconfig.')
 #provides=(${pkgbase}=${_basekernel} 'aufs3')	# for $pkgname-optimized
 provides=(${pkgbase}=${_basekernel} 'aufs3')
 # below 'provides' is for when you have no other kernel (which is a bad idea anyway)
 # provides=(${pkgbase}=${_basekernel} 'linux=${pkgver}' 'aufs3')
 # If generic build, then conflict with all optimized ones
 conflicts=()
 for _cpusuffix in $_CPUSUFFIXES ; do
   conflicts+=(${pkgbase}-${_cpusuffix}) 
 done  
 replaces=('kernel26-pf' 'aufs3')
 backup=("etc/mkinitcpio.d/${pkgbase}.preset")
 install='linux.install'

 #'
  cd "${srcdir}/linux-${_basekernel}"

  # Remove undeeded aufs3 git tree
  rm -fr aufs3 2>/dev/null

  # work around the AUR parser
  # This allows building cpu-optimized packages with according package names.
  # Useful for repo maintainers.
  headers="headers"
  pkgnameopt="${pkgbase}"		# this MUST be outside the following 'if'
  if [[ "$_PKGOPT" = "y" ]]; then	# package naming according to optimization
    case $CPU in
     CORE2)
         pkgname="${pkgbase}-core2"
         pkgdesc="${_pkgdesc} Intel Core2 optimized."
         ;;
     K7)
        pkgname="${pkgbase}-k7"
        pkgdesc="${_pkgdesc} AMD K7 optimized."
        ;;
     K8)
         pkgname="${pkgbase}-k8" 
         pkgdesc="${_pkgdesc} AMD K8 optimized."
	 ;;
     K10)
         pkgname="${pkgbase}-k10"
	 pkgdesc="§{_pkgdesc} AMD K10 optimized"
         ;;
     BARCELONA)
         pkgname="${pkgbase}-barcelona"
         pkgdesc="${_pkgdesc} AMD Barcelona optimized."
	 ;;
     BOBCAT)
	 pkgname="${pkgbase}-bobcat"
	 pkgdesc="${_pkgdesc} AMD Bobcat optimized."
	 ;;
     BULLDOZER)
	 pkgname="${pkgbase}-bulldozer"
	 pkgdesc="${_pkgdesc} AMD Bulldozer optimized."
	 ;;
     PILEDRIVER)
	 pkgname="${pkgbase}-piledriver"
	 pkgdesc="${_pkgdesc} AMD Piledriver optimized."
	 ;;
     PSC)
         pkgname="${pkgbase}-psc"
         pkgdesc="${_pkgdesc} Intel Pentium4/D/Xeon optimized."
         ;;
     ATOM)
         pkgname="${pkgbase}-atom"
         pkgdesc="${_pkgdesc} Intel Atom optimized."
         ;;
     PENTIUMII)
         pkgname="${pkgbase}-p2"
         pkgdesc="${_pkgdesc} Intel Pentium2 optimized."
         ;;
     PENTIUMIII)
         pkgname="${pkgbase}-p3"
         pkgdesc="${_pkgdesc} Intel Pentium3 optimized."
         ;;
     PENTIUMM)
         pkgname="${pkgbase}-pm"
         pkgdesc="${_pkgdesc} Intel Pentium-M optimized."
         ;;
     PENTIUM4)
         pkgname="${pkgbase}-p4"
         pkgdesc="${_pkgdesc} Intel Pentium4 optimized."
         ;;
     NEHALEM)
	 pkgname="${pkgbase}-nehalem"
         pkgdesc="${_pkgdesc} Intel Core Nehalem optimized."
	 ;;
     SANDYBRIDGE)
         pkgname="${pkgbase}-sandybridge"
         pkgdesc="${_pkgdesc} Intel 2nd Gen Core processors including Sandy Bridge."
	 ;;
     IVYBRIDGE)
         pkgname="${pkgbase}-ivybridge"
         pkgdesc="${_pkgdesc} Intel 3rd Gen Core processors including Ivy Bridge."
	 ;;
     HASWELL)
         pkgname="${pkgbase}-haswell"
         pkgdesc="${_pkgdesc} 4th Gen Core processors including Haswell."
	 ;;
     BROADWELL)
         pkgname="${pkgbase}-broadwell"
         pkgdesc="${_pkgdesc} 5th Gen Core processors including Broadwell."
	 ;;
     SILVERMONT)
         pkgname="${pkgbase}-silvermont"
         pkgdesc="${_pkgdesc} 6th Gen Core processors including Silvermont."
	 ;;
     default)
  # Note to me: DO NOT EVER REMOVE THIS. It's for the AUR PKGBUILD parser.
         pkgname="${pkgbase}"
         pkgdesc="Linux kernel and modules with the pf-kernel patch [-ck patchset (BFS included), TuxOnIce, BFQ] and aufs3"
         ;;
    esac


  # If optimized build, conflict with generic and other optimized ones

  if [[ "$pkgname" != "$pkgbase" ]]; then
	pkgnameopt="${pkgname}"		# this MUST be inside this if-fi
	pkgname="${pkgbase}"
	echo pkgname $pkgname
	cpuopt=`sed -e "s/linux-pf-//" <<<$pkgnameopt`		# get suffix
	cpuoptdesc=`sed -e "s/${_pkgdesc}//" <<<$pkgdesc`	# get description
	conflicts=(${conflicts[@]/linux-pf-${cpuopt}/})		# remove current
	conflicts=(${conflicts[@]/linux-pf-headers-${cpuopt}/})	# remove current's headers
	export cpuopt cpuoptdesc
  fi

  # second batch check ends here
 fi

 # Pass conflicts array to linux-pf-headers() BEFORE adding generic linux-pf or headers will conflict
 export _conflicts=${conflicts[@]}
 conflicts=('linux-pf' 'kernel26-pf' ${conflicts[@]})	# add generic packages

  echo
  echo "    ========================================"
  msg  "The packages will be named ${pkgnameopt} and"
  if [[ "$cpuopt" ]]; then
       msg  "and ${pkgbase}-${headers}-${cpuopt}"
  else
       msg  "and ${pkgbase}-${headers}"
  fi
  msg  "${pkgdesc}"
  echo "    ========================================"
  echo

 ### package_linux-pf

  cd "${srcdir}/linux-${_basekernel}"

  KARCH=x86

  echo # get kernel version
   _kernver="$(make LOCALVERSION= kernelrelease)"


  ### c/p from linux-ARCH

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"


  # install fallback mkinitcpio.conf file and preset file for kernel
  install -D -m644 "${srcdir}/linux.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # set correct depmod command for install
  #sed \
  #  -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/" \
  #  -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/" \
  #  -i "${startdir}/linux.install"
   sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgbase}\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgbase}.img\"|" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgbase}-fallback.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

  # Now we call depmod...
  depmod -b "$pkgdir" -F System.map "$_kernver"
  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "$pkgdir/lib" "$pkgdir/usr"
  # add vmlinux
  install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/build/${_kernver}/vmlinux"

# end c/p

  ###
  # Trick the AUR parser to accept split PKGBUILD
  true && pkgname="${pkgnameopt}"
}

### package_linux-pf-headers
package_linux-pf-headers() {
  pkgdesc="Header files and scripts for building modules for linux-pf kernel."
  depends=('linux-pf') 
  conflicts=( ${_conflicts[@]} )
  for _cpusuffix in $_CPUSUFFIXES ; do
    conflicts+=(linux-pf-headers-$_cpusuffix)
  done
  [[ ${cpuopt} ]] && pkgname=${pkgname}-${cpuopt} && depends=${depends}-${cpuopt}
  [[ ${cpuoptdesc} ]] && pkgdesc=${pkgdesc}${cpuoptdesc}
  provides=('linux-pf-headers')
  cd "${srcdir}/linux-${_basekernel}"
# c/p from linux-ARCH

  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"


  cd "${srcdir}/${_srcname}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

  for i in acpi asm-generic config crypto drm generated keys linux math-emu \
    media net pcmcia scsi sound trace uapi video xen; do
    cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
  done

  # copy arch includes for external modules
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
  cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

  if [ "${CARCH}" = "i686" ]; then
    cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
  fi

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"

  # add dm headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
  cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

  # add inotify.h
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
  cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

  # add wireless headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
  cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core"
  cp drivers/media/dvb-core/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core/"
  # and...
  # http://bugs.archlinux.org/task/11194
  ###
  ### DO NOT MERGE OUT THIS IF STATEMENT
  ### IT AFFECTS USERS WHO STRIP OUT THE DVB STUFF SO THE OFFICIAL ARCH CODE HAS A CP
  ### LINE THAT CAUSES MAKEPKG TO END IN AN ERROR
  ###
  if [ -d include/config/dvb/ ]; then
      mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
      cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
  fi
  
  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
  cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

  # add dvb headers
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
  cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
  cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
  cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

  # add xfs and shmem for aufs building
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"
  # removed in 3.17 series
  # cp fs/xfs/xfs_sb.h "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs/xfs_sb.h"

  # copy in Kconfig files
  for i in $(find . -name "Kconfig*"); do
    mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
    cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
  done

  chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
  find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;
  
  # strip scripts directory
  find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
      *application/x-sharedlib*) # Libraries (.so)
        /usr/bin/strip ${STRIP_SHARED} "${binary}";;
      *application/x-archive*) # Libraries (.a)
        /usr/bin/strip ${STRIP_STATIC} "${binary}";;
      *application/x-executable*) # Binaries
        /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
  done
  
  # remove unneeded architectures
  rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arc,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
  # end c/p
  
  # Add version.h for dumb binary blob installers which aren't
  cd "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"
  [[ -e version.h ]] || ln -s ../generated/uapi/linux/version.h
  
}

# Work around the AUR parser
pkgdesc="Linux kernel and modules with the pf-kernel patch [-ck patchset (BFS included), TuxOnIce, BFQ] and aufs3"

# makepkg -g >>PKGBUILD
sha256sums=('caf51f085aac1e1cea4d00dbbf3093ead07b551fc07b31b2a989c05f8ea72d9f'
            '30e86da2fd10b5750371563546b92c6a0017c8ae78bbb3c067d3125deef1f048'
            'fe834ce67939cf6d08e76a0f5fa58cf98d0cfffffabed47142c69bcca0cc9219'
            '82d660caa11db0cd34fd550a049d7296b4a9dcd28f2a50c81418066d6e598864'
            '1256b241cd477b265a3c2d64bdc19ffe3c9bbcee82ea3994c590c2c76e767d99'
            '247c7e2001e4fe28e8cabd0f861240ac13487a942bcb2dbd92ea50302d799414'
            'SKIP'
            '64330f2b455c1b5be9f595e00a111ed6edfe9eec010453ccb6e4d7dfe3cdad3f')
