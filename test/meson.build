# Copyright 2021-2022 David Robillard <d@drobilla.net>
# SPDX-License-Identifier: CC0-1.0 OR ISC

autoship = find_program('autoship', required: false)
run_test_suite = files('run_test_suite.py')

wrapper = meson.get_external_property('exe_wrapper', '')

####################
# Project Metadata #
####################

if autoship.found()
  test('autoship', autoship, args: ['test', serd_src_root], suite: 'data')
endif

##############
# Unit Tests #
##############

unit_tests = [
  'env',
  'free_null',
  'node',
  'read_chunk',
  'reader_writer',
  'string',
  'uri',
  'writer',
  'mdb'
]

foreach unit : unit_tests
  test(
    unit,
    executable(
      'test_@0@'.format(unit),
      files('test_@0@.c'.format(unit)),
      c_args: c_suppressions,
      dependencies: serd_dep,
    ),
    suite: 'unit',
  )
endforeach

################
# System Tests #
################

if not get_option('tools').disabled()
  if wrapper != ''
    script_args = ['--wrapper', wrapper, '--serdi', serdi.full_path()]
  else
    script_args = ['--serdi', serdi.full_path()]
  endif

  serd_ttl = files('../serd.ttl')[0]

  test('serd.ttl', serdi, args: [serd_ttl], suite: 'data')

  # Command line options

  good_args = [
    ['-v'],
    ['-h'],
    ['-s', '<urn:eg:s> a <urn:eg:T> .'],
  ]

  foreach args : good_args
    test(args[0], serdi, args: args, suite: ['serdi', 'options'])
  endforeach

  bad_args = [
    ['/no/such/file'],
    ['ftp://unsupported.org'],
    ['-c'],
    ['-i', 'unknown'],
    ['-i', 'turtle'],
    ['-i'],
    ['-fi'],
    ['-o', 'unknown'],
    ['-o'],
    ['-p'],
    ['-r'],
    ['-z'],
    ['-s', '<foo> a <Bar> .'],
  ]

  foreach args : bad_args
    name = ' '.join(args).underscorify()
    test(name, serdi,
         args: args,
         should_fail: true,
         suite: ['serdi', 'options'])
  endforeach

  test('none', serdi, should_fail: true, suite: ['serdi', 'options'])

  test('quiet', files('test_quiet.py'),
       args: script_args + files('bad/bad-base.ttl'),
       suite: ['serdi', 'options'])

  # Inputs

  test('stdin', files('test_stdin.py'),
       args: script_args,
       suite: ['serdi', 'input'])

  test('string', serdi,
       args: ['-s', '<foo> a <Bar> .'],
       should_fail: true,
       suite: ['serdi', 'input'])

  test('missing', serdi,
       args: ['-i', 'turtle'],
       should_fail: true,
       suite: ['serdi', 'input'])

  test('no_such_file', serdi,
       args: ['no_such_file'],
       should_fail: true,
       suite: ['serdi', 'input'])

  test('remote', serdi,
       args: ['ftp://example.org/unsupported.ttl'],
       should_fail: true,
       suite: ['serdi', 'input'])

  # IO errors

  test('read_dir', serdi,
       args: ['-e', 'file://@0@/'.format(serd_src_root)],
       should_fail: true,
       suite: 'io_errors')

  test('bulk_read_dir', serdi,
       args: ['file://@0@/'.format(serd_src_root)],
       should_fail: true,
       suite: 'io_errors')

  test('write_error', files('test_write_error.py'),
       args: script_args + [serd_ttl],
       suite: 'io_errors')

  # RDF test suites

  ## Serd-specific test suites

  serd_suites = ['good', 'bad']
  serd_base = 'http://drobilla.net/sw/serd/test/'

  ### Run all suites with no special arguments
  foreach name : serd_suites
    manifest = files(name / 'manifest.ttl')
    base_uri = serd_base + name + '/'
    test(name, run_test_suite,
         args: script_args + [manifest, base_uri],
         suite: ['rdf', 'serd'],
         timeout: 240)
  endforeach

  ### The lax suite is special because it is run twice...
  lax_manifest = files('lax/manifest.ttl')
  lax_base_uri = serd_base + name + '/'

  ### ... once with strict parsing to test the hard errors
  test('lax.strict', run_test_suite,
       args: script_args + [lax_manifest, lax_base_uri],
       is_parallel: false,
       suite: ['rdf', 'serd'],
       timeout: 240)

  ### ... and once with lax parsing to tolerate them
  test('lax.lax', run_test_suite,
       args: script_args + [lax_manifest, lax_base_uri, '--', '-l'],
       is_parallel: false,
       suite: ['rdf', 'serd'],
       timeout: 240)

  ## Standard W3C test suites

  w3c_suites = ['Turtle', 'NTriples', 'NQuads', 'TriG']
  w3c_base = 'http://www.w3.org/2013/'

  foreach syntax : w3c_suites
    manifest = files(syntax + 'Tests' / 'manifest.ttl')
    base_uri = w3c_base + syntax + 'Tests/'

    args = ['--syntax', syntax, manifest, base_uri]
    if syntax == 'TriG'
      args += ['--', '-a']
    endif

    test(syntax, run_test_suite,
         args: script_args + args,
         suite: ['rdf', 'w3c'],
         timeout: 240)
  endforeach
endif
