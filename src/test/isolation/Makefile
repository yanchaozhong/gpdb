#
# Makefile for isolation tests
#

subdir = src/test/isolation
top_builddir = ../../..
include $(top_builddir)/src/Makefile.global

# where to find psql for testing an existing installation
PSQLDIR = $(bindir)

override CPPFLAGS := -I$(srcdir) -I$(libpq_srcdir) -I$(srcdir)/../regress $(CPPFLAGS)

OBJS =  specparse.o isolationtester.o

gpstringsubs.pl:
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/gpstringsubs.pl

gpdiff.pl: atmsort.pm explain.pm GPTest.pm
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/gpdiff.pl

GPTest.pm:
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/GPTest.pm

atmsort.pm:
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/atmsort.pm

explain.pm:
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/explain.pm

submake-regress:
	$(MAKE) -C $(top_builddir)/src/test/regress pg_regress.o

pg_regress.o: | submake-regress
	rm -f $@ && $(LN_S) $(top_builddir)/src/test/regress/pg_regress.o .

pg_isolation_regress: isolation_main.o pg_regress.o
	$(CC) $(CFLAGS) $^ $(LDFLAGS) $(LDFLAGS_EX) $(LIBS) -o $@$(X)

all: isolationtester pg_isolation_regress

isolationtester: $(OBJS) | submake-libpq submake-libpgport
	$(CC) $(CFLAGS) $^ $(libpq_pgport) $(LDFLAGS) $(LDFLAGS_EX) $(LIBS) -o $@$(X)

distprep: specparse.c specscanner.c

# There is no correct way to write a rule that generates two files.
# Rules with two targets don't have that meaning, they are merely
# shorthand for two otherwise separate rules.  To be safe for parallel
# make, we must chain the dependencies like this.  The semicolon is
# important, otherwise make will choose the built-in rule for
# gram.y=>gram.c.

all: isolationtester$(X) pg_isolation_regress$(X)

specparse.h: specparse.c ;

# specscanner is compiled as part of specparse
specparse.o: specscanner.c

specparse.c: specparse.y
ifdef BISON
	$(BISON) $(BISONFLAGS) -o $@ $<
else
	@$(missing) bison $< $@
endif

specscanner.c: specscanner.l
ifdef FLEX
	$(FLEX) $(FLEXFLAGS) -o'$@' $<
else
	@$(missing) flex $< $@
endif

# specparse.c and specscanner.c are in the distribution tarball,
# so do not clean them here
clean distclean:
	rm -f isolationtester$(X) pg_isolation_regress$(X) $(OBJS) isolation_main.o
	rm -f pg_regress.o
	rm -f gpstringsubs.pl gpdiff.pl atmsort.pm explain.pm GPTest.pm
	rm -rf $(pg_regress_clean_files)

maintainer-clean: distclean
	rm -f specparse.c specscanner.c

installcheck: all gpdiff.pl gpstringsubs.pl
	./pg_isolation_regress --psqldir='$(PSQLDIR)' --inputdir=$(srcdir) --schedule=$(srcdir)/isolation_schedule

# Version of the install check test that includes the prepared_transactions test
# It only makes sense to run this if set up to use prepared transactions,
# via the postgresql.conf.
installcheck-prepared-txns: all
	./pg_isolation_regress --psqldir='$(PSQLDIR)' --inputdir=$(srcdir) --schedule=$(srcdir)/isolation_schedule prepared-transactions

check: all
	./pg_isolation_regress --temp-install=./tmp_check --inputdir=$(srcdir) --top-builddir=$(top_builddir) --schedule=$(srcdir)/isolation_schedule
