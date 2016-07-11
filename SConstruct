# Starter SConstruct for enscons

import sys
from distutils import sysconfig
import pytoml as toml
import enscons

metadata = dict(toml.load(open('pyproject.toml')))['tool']['enscons']

# most specific binary, non-manylinux1 tag should be at the top of this list
import wheel.pep425tags
for tag in wheel.pep425tags.get_supported():
    full_tag = '-'.join(tag)
    if not 'manylinux' in tag:
        break

# full_tag = py2.py3-none-any # pure Python packages compatible with 2+3

env = Environment(tools=['default', 'packaging', enscons.generate],
                  PACKAGE_METADATA=metadata,
                  WHEEL_TAG=full_tag,
                  ROOT_IS_PURELIB=False)

# Only *.py is included automatically by setup2toml.
# Add extra 'purelib' files or package_data here.
py_source = ['hello_pyrust.py']

rust_libname = 'libhello_pyrust' + env['SHLIBSUFFIX']
rust_lib = 'rust/target/release/' + rust_libname

# Build rust
env.Command(
        target=rust_lib,
        source=["rust/Cargo.toml", "rust/src/hello_pyrust.h", "rust/src/lib.rs"],
        action="cargo build --release", 
        chdir="rust"
        )

# Copy compiled library into base directory
local_rust = env.Command(
        target=rust_libname,
        source=rust_lib,
        action=Copy('$TARGET', '$SOURCE'))
        
local_rust_h = env.Command(
        target='hello_pyrust.h',
        source='rust/src/hello_pyrust.h',
        action=Copy('$TARGET', '$SOURCE'))

env.Whl('platlib', py_source + local_rust + local_rust_h, root='')

# Add automatic source files, plus any other needed files.
sdist_source=FindSourceFiles() + ['PKG-INFO', 'setup.py', 'LICENSE', 'README.md']

sdist = env.Package(
        NAME=env['PACKAGE_NAME'],
        VERSION=env['PACKAGE_METADATA']['version'],
        PACKAGETYPE='src_targz',
        source=sdist_source,
        target=['/'.join(['dist', env['PACKAGE_NAME'] + '-' + env['PACKAGE_VERSION'] + '.tar.gz'])],
        )
