# sqlite3

Install Guide to add SQLITE3 version 3.9.0 to python2.6 on Linux.


1. GETTING STARTED

su -
export JQLITE="$HOME/bin/jqlite"
mkdir -p $JQLITE
cd $JQLITE

curl 'https://www.sqlite.org/src/tarball/sqlite.tar.gz?ci=trunk' | tar xz
mv sqlite/* .


2.COMPILING SQLITE WITH JSON1 AND FTS5

export CFLAGS="-DSQLITE_ENABLE_COLUMN_METADATA \
-DSQLITE_ENABLE_DBSTAT_VTAB \
-DSQLITE_ENABLE_FTS3 \
-DSQLITE_ENABLE_FTS3_PARENTHESIS \
-DSQLITE_ENABLE_FTS4 \
-DSQLITE_ENABLE_FTS5 \
-DSQLITE_ENABLE_JSON1 \
-DSQLITE_ENABLE_STAT4 \
-DSQLITE_ENABLE_UPDATE_DELETE_LIMIT \
-DSQLITE_SECURE_DELETE \
-DSQLITE_SOUNDEX \
-DSQLITE_TEMP_STORE=3 \
-O2 \
-fPIC"

LIBS="-lm" ./configure --prefix=$JQLITE --enable-static --enable-shared --enable-fts5 --enable-json1
make
make install


3. BUILDING PYSQLITE

git clone https://github.com/ghaering/pysqlite
cd pysqlite/
cp $JQLITE/sqlite3.c ./
echo -e "library_dirs=$JQLITE/lib" >> setup.cfg
echo -e "include_dirs=$JQLITE/include" >> setup.cfg
LIBS="-lm" python setup.py build_static
python setup.py install
ln -s $JQLITE/bin/sqlite3 $JQLITE/sqlite

4. COPY PYSQLITE BINARY
Additional, you must copy builded pysqlite2 files(*.py, _sqlite.so) into system pysqlite2 directory.
NOTE: If you installed Python 2.6 on your machine, set lib.linux-x86_64-2.6 instead of lib.linux-x86_64-2.7 for the path.

cp -rf $JQLITE/pysqlite/build/lib.linux-x86_64-2.7/pysqlite2/* /usr/share/pyshared/pysqlite2/
cp -rf $JQLITE/pysqlite/build/lib.linux-x86_64-2.7/pysqlite2/* /usr/local/lib/python2.7/dist-packages/sqlalchemy/dialects/sqlite/
cp -rf $JQLITE/pysqlite/build/lib.linux-x86_64-2.7/pysqlite2/* /usr/local/lib/python2.7/dist-packages/pysqlite2/


5. BUILDING APSW

cd $JQLITE
git clone https://github.com/rogerbinns/apsw
cd apsw
cp $JQLITE/sqlite3{ext.h,.h,.c} ./
echo -e "library_dirs=$JQLITE/lib" >> setup.cfg
echo -e "include_dirs=$JQLITE/include" >> setup.cfg
LIBS="-lm" python setup.py build
python setup.py install


6. SAMPLE CODE

python

from pysqlite2 import dbapi2 as sqlite
import json
dbsession = sqlite.connect(':memory:')
dbsession.execute("CREATE VIRTUAL TABLE IF NOT EXISTS searchcontent USING fts5(name,price,count)")
dbsession.execute('INSERT INTO searchcontent(rowid,name,price,count) VALUES(1,\'camera\',\'$2000\',\'150\')');
dbsession.execute('INSERT INTO searchcontent(rowid,name,price,count) VALUES(2,\'mic\',\'$300\',\'90\')');
dbsession.execute('SELECT * FROM searchcontent');
dbsession.execute('SELECT json(?)', (1337,)).fetchone()
json_string = json.dumps(dbsession.execute('SELECT json(?)', (1337,)).fetchone())
print json_string

7. NOTE

If don't make reference sqlite when reboot your machine, run the command as follow.

echo -e "export PATH=$PATH:/root/bin/jqlite/bin" >> /root/.bashrc
echo -e "export PATH=$PATH:/root/bin/jqlite/bin" >> /etc/profile
source /root/.bashrc
source /etc/profile

If you reinstall Python and SQLITE3, please reference "https://www.bluebill.net/2016/04/24/install-python-and-sqlite-from-source/"


8. THANK YOU.
