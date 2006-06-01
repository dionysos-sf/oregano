import os;
import SCons;

# APP Version #
VERSION = '0.50.0'

# Command line options #
opts = Options ('oregano.conf');
opts.Add (PathOption ('PREFIX', 'System base prefix path', '/usr/local'));
opts.Add (PackageOption ('DESTDIR', 'System base installation path', '/'));

# Dependencies #
deps = []
deps.append ({'lib': 'gtk+-2.0',            'ver': '2.8.0'})
deps.append ({'lib': 'libglade-2.0',        'ver': '2.5.0'})
deps.append ({'lib': 'libgnomeui-2.0',      'ver': '2.12.0'})
deps.append ({'lib': 'libgnomeprint-2.2',   'ver': '2.12.0'})
deps.append ({'lib': 'libxml-2.0',          'ver': '2.6.0'})
deps.append ({'lib': 'libgnomecanvas-2.0',  'ver': '2.12.0'})
deps.append ({'lib': 'gtksourceview-1.0',   'ver': '1.0'})
deps.append ({'lib': 'cairo',               'ver': '1.0.0'})
deps.append ({'lib': 'libgnomeprintui-2.2', 'ver': '2.12.0'})

# CUSTOM CHECK FUNC #
def CheckPkg (context, pkg, version):
	msg = 'Checking for pkg %s >= %s ... ' % (pkg, version)
	context.Message ('   %-60s ' % (msg));
	result = os.system ('pkg-config --exists \'%s >= %s\'' % (pkg, version) );
	if result == 0:
		result = 'yes';
	else:
		result = 0;
	context.Result(result)
	return result;

CEnv = Environment (options = opts);
CEnv.SourceSignatures('timestamp')

# po_helper
# #
# # this is not a builder. we can't list the .po files as a target,
# # because then scons -c will remove them (even Precious doesn't alter
# # this). this function is called whenever a .mo file is being
# # built, and will conditionally update the .po file if necessary.
# #
def po_helper(po,pot):
	args = [ 'msgmerge',
		'--update',
		po,
		pot,
	]
	print 'Updating ' + po
	return os.spawnvp (os.P_WAIT, 'msgmerge', args)

# mo_builder: builder function for (binary) message catalogs (.mo)
#
# first source:  .po file
# second source: .pot file
#
def mo_builder(target,source,env):
	# Do not update .po files, just compile it
	# po_helper (source[0].get_path(), source[1].get_path())
	args = [ 'msgfmt',
		'-c',
		'-o',
		target[0].get_path(),
		source[0].get_path()
	]
	return os.spawnvp (os.P_WAIT, 'msgfmt', args)

mo_bld = Builder (action = mo_builder)

CEnv.Append (BUILDERS = {'MoBuild' : mo_bld})
CEnv.Append (CCFLAGS = Split ('-Wall'));

# Check dependencies #
if not CEnv.GetOption ('clean'):
	Cconf = Configure (CEnv, custom_tests = {'CheckPkg' : CheckPkg })

	for dep in deps:
		if not Cconf.CheckPkg (dep['lib'], dep['ver']): Exit(1);

	Cconf.Finish ();

# Create compiler command line from dependencies #
for dep in deps:
	CEnv.ParseConfig ('pkg-config --cflags --libs '+dep['lib']);

# Configure environment #
DataDir = CEnv['PREFIX']+'/share'
OreganoDir = DataDir+'/oregano'

CEnv['DATADIR'] = DataDir
CEnv['OREGANODIR'] = OreganoDir
CEnv['VERSION'] = VERSION
CEnv['DOMAIN'] = "oregano"
CEnv['POTFILE'] = "oregano.pot"

Export ('CEnv')

SConscript ('src/SConscript');
SConscript ('data/SConscript');
SConscript ('po/SConscript');

# Install Target #
CEnv.Alias ('install', CEnv.Command ('oregano.keys', 'oregano.keys.in', "sed 's/@icondir@/"+os.path.join (CEnv['DESTDIR'], CEnv['DATADIR'],"mine-info").replace ("/", "\\/")+"/' < $SOURCE > $TARGET"))
CEnv.Alias ('install', CEnv.Command ('oregano.xml', 'oregano.xml.in', "cp $SOURCE $TARGET"))
CEnv.Alias('install', CEnv.Install (os.path.join (CEnv['DESTDIR'], CEnv['DATADIR'], 'mime/packages'), Split('oregano.xml')))
CEnv.Alias('install', CEnv.Install (os.path.join (CEnv['DESTDIR'], CEnv['DATADIR'], 'mime-info'), Split('oregano.mime oregano.keys')))
CEnv.Alias('install', CEnv.Install (os.path.join (CEnv['DESTDIR'], CEnv['DATADIR'], 'gnome/apps/Applications'), Split('oregano.desktop')))

# Update mime database #
CEnv.Alias ('install', CEnv.Command ('update-mime-database', 'oregano.xml', "update-mime-database "+os.path.join (CEnv['DESTDIR'], CEnv['DATADIR'], 'mime')))

