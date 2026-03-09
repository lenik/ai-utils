# ai-utils — Ascii Art format utilities (encode/decode in ascii art)
# Refactored from Autotools to Meson.

project('ai-utils', 'c',
  version : '2.2.0',
  license : 'AGPL-3.0-or-later',
  default_options : ['warning_level=1', 'c_std=c11'])

cc = meson.get_compiler('c')

# ---- Dependencies ----
glib_dep = dependency('glib-2.0', required : true)

# Paths (match Autotools layout)
prefix = get_option('prefix')

# libbas via pkg-config only (requires libbas-c-dev installed)
bas_dep = dependency('bas-c', required : true)
datadir = prefix / get_option('datadir')
libdir = prefix / get_option('libdir')
libprefix = prefix / 'lib'
bindir = prefix / get_option('bindir')
mandir = prefix / get_option('mandir')
sysconfdir = get_option('sysconfdir')
if sysconfdir == ''
  sysconfdir = prefix / 'etc'
endif
pkgdatadir = datadir / 'ai-utils'
templatedir = pkgdatadir / 'template'
completiondir = datadir / 'bash-completion' / 'completions'
setupdir = datadir / 'setup' / 'ai-utils'
shlibdir = libprefix / 'shlib.d'

# ---- config.h ----
conf = configuration_data()
conf.set_quoted('PACKAGE', 'ai-utils')
conf.set_quoted('PACKAGE_NAME', 'ai-utils')
conf.set_quoted('PACKAGE_STRING', 'ai-utils ' + meson.project_version())
conf.set_quoted('PACKAGE_VERSION', meson.project_version())
conf.set_quoted('PACKAGE_BUGREPORT', 'xjl-bug@99jsj.com')
conf.set_quoted('VERSION', meson.project_version())
conf.set_quoted('TEMPLATE_DIR', templatedir)
conf.set_quoted('BUILD_DATE', run_command('date', '+%Y-%m-%d', check : true).stdout().strip())
conf.set_quoted('DATADIR', datadir)
conf.set('STDC_HEADERS', 1)
conf.set('HAVE_STRCHR', cc.has_function('strchr'))
conf.set('HAVE_STRTOL', cc.has_function('strtol'))
foreach h : ['stdlib.h', 'stdio.h', 'string.h', 'strings.h', 'inttypes.h', 'stdint.h', 'unistd.h', 'sys/stat.h', 'sys/types.h']
  conf.set('HAVE_' + h.underscorify().to_upper(), cc.has_header(h))
endforeach

configure_file(output : 'config.h', configuration : conf)

# ---- Executables ----
# Note: Requires libbas-c installed (meson install from lib/libbas-c) for bas/*.h layout.
common_src = files('ai.h', 'aicodec.c')
inc = include_directories('.')
# strdup, mkdtemp, symlink, tempnam need _GNU_SOURCE on glibc; LOGGER_NAME required by bas/cli
c_extra = ['-DLOG_LEVEL=4', '-D_DEFAULT_SOURCE']

executable('ai',
  'ai.c', common_src,
  include_directories : [inc],
  dependencies : [glib_dep, bas_dep],
  c_args : c_extra + ['-DLOGGER_NAME=ai'],
  install : true,
)

executable('air',
  'air.c', common_src,
  include_directories : [inc],
  dependencies : [glib_dep, bas_dep],
  c_args : c_extra + ['-DLOGGER_NAME=air'],
  install : true,
)

executable('unai',
  'unai.c', common_src,
  include_directories : [inc],
  dependencies : [glib_dep, bas_dep],
  c_args : c_extra + ['-DLOGGER_NAME=unai'],
  install : true,
)

# ---- loadimage: shared library (bash loadable builtin) ----
bash_dep = dependency('bash', required : true)
bash_inc = bash_dep.get_variable(pkgconfig : 'includedir')
bash_inc_dirs = [
  include_directories(bash_inc / 'bash', is_system : true),
  include_directories(bash_inc / 'bash' / 'include', is_system : true),
]
# Bash loadable builtin: symbols (get_string_value, builtin_usage, etc.) come from bash at dlopen
shared_library('loadimage',
  'loadimage.c', common_src,
  include_directories : [inc] + bash_inc_dirs,
  dependencies : [bash_dep, bas_dep],
  c_args : ['-fPIC', '-DSHELL', '-D_GNU_SOURCE', '-DLOGGER_NAME=loadimage'],
  link_args : ['-Wl,--unresolved-symbols=ignore-all'],
  install : true,
  install_dir : libdir,
)

# ---- Data and scripts ----
# Completions (with @templatedir@; use set() so shell gets /path not ""/path"")
conf_comp = configuration_data()
conf_comp.set('templatedir', templatedir)
ai_utils = configure_file(
  input : 'completions/ai_utils.in',
  output : 'ai_utils',
  configuration : conf_comp,
)
install_data(ai_utils,
  install_dir : completiondir,
  install_mode : 'rw-r--r--',
)
install_symlink('ai', pointing_to : 'ai_utils', install_dir : completiondir)
install_symlink('unai', pointing_to : 'ai_utils', install_dir : completiondir)

# aifmt script (@bindir@) — used by aifmt.service or manually
conf_aifmt = configuration_data()
conf_aifmt.set_quoted('bindir', bindir)
aifmt = configure_file(input : 'aifmt.in', output : 'aifmt', configuration : conf_aifmt)
install_data(aifmt, install_dir : pkgdatadir, install_mode : 'rwxr-xr-x')

# aifmt systemd service (replaces sysvinit)
systemd_dep = dependency('systemd', required : true)
systemdsystemunitdir = systemd_dep.get_variable(pkgconfig : 'systemdsystemunitdir')
conf_svc = configuration_data()
conf_svc.set_quoted('pkgdatadir', pkgdatadir)
aifmt_svc = configure_file(
  input : 'aifmt.service.in',
  output : 'aifmt.service',
  configuration : conf_svc,
)
install_data(
    aifmt_svc, 
    # install_dir : systemdsystemunitdir, 
    install_dir : libprefix / 'systemd' / 'system',
    install_mode : 'rw-r--r--')

# Setup scripts (postinst, prerm)
conf_setup = configuration_data()
conf_setup.set_quoted('sysconfdir', sysconfdir)
conf_setup.set_quoted('pkgdatadir', pkgdatadir)
conf_setup.set_quoted('bindir', bindir)
postinst = configure_file(input : 'postinst.in', output : 'postinst', configuration : conf_setup)
prerm = configure_file(input : 'prerm.in', output : 'prerm', configuration : conf_setup)
install_data(postinst, install_dir : setupdir, install_mode : 'rwxr-xr-x')
install_data(prerm, install_dir : setupdir, install_mode : 'rwxr-xr-x')

# Template directory (keep template/ so debian/ai-utils.install usr/share/ai-utils/template/ matches)
install_subdir('template', install_dir : pkgdatadir, strip_directory : false)

# shlib.d/ai (enable -f loadimage loadimage)
install_data('shlib.d/ai', install_dir : shlibdir, install_mode : 'rw-r--r--')

# Man pages
install_man(['ai.1', 'air.1', 'unai.1'])
