#
# libspice-common
#
spice_common_sources = [
  'agent.c',
  'agent.h',
  'backtrace.c',
  'backtrace.h',
  'canvas_utils.c',
  'canvas_utils.h',
  'demarshallers.h',
  'draw.h',
  'lines.c',
  'lines.h',
  'log.c',
  'log.h',
  'lz.c',
  'lz.h',
  'lz_common.h',
  'lz_config.h',
  'macros.h',
  'marshaller.c',
  'marshaller.h',
  'mem.c',
  'mem.h',
  'messages.h',
  'pixman_utils.c',
  'pixman_utils.h',
  'quic.c',
  'quic.h',
  'quic_config.h',
  'rect.h',
  'region.c',
  'region.h',
  'ring.h',
  'rop3.c',
  'rop3.h',
  'snd_codec.c',
  'snd_codec.h',
  'utils.c',
  'utils.h',
  'verify.h',
  'recorder.h',
  'sm2-EVP/c/sm2.h',
  'sm2-EVP/c/sm2.c',
  'sm2-EVP/cpp/sm2.h',
  'sm2-EVP/cpp/sm2.cpp',
]

if get_option('instrumentation') == 'recorder'
  spice_common_sources += [
    'recorder/recorder.c',
    'recorder/recorder.h',
    'recorder/recorder_ring.c',
    'recorder/recorder_ring.h'
  ]
endif
if get_option('instrumentation') == 'agent'
  spice_common_sources += [
    'agent_interface.c',
    'agent_interface.h'
  ]
endif

spice_common_lib = static_library('spice-common', spice_common_sources,
                                  install : false,
                                  include_directories : spice_common_include,
                                  dependencies : spice_common_deps)

spice_common_dep = declare_dependency(link_with : spice_common_lib,
                                      include_directories : spice_common_include,
                                      dependencies : spice_common_deps)


# client_demarshallers
if spice_common_generate_client_code or spice_common_generate_server_code
  codegen_cmd = [python, spice_codegen]
  codegen_args = ['--generate-demarshallers',
                  '--client',
                  '--include', 'common/messages.h',
                  '--generated-declaration-file', '@OUTPUT1@',
                  '@INPUT@', '@OUTPUT0@']

  client_demarshallers = custom_target('client_demarshallers',
                                       input : [spice_proto],
                                       output : ['generated_client_demarshallers.c', 'generated_messages.h'],
                                       install : false,
                                       command : [codegen_cmd, codegen_args],
                                       depend_files : [spice_codegen_files, 'messages.h'])
endif

#
# libspice-common-client
#
if spice_common_generate_client_code
  # client_marshallers
  codegen_args = ['--generate-marshallers',
                  '--generate-header',
                  '-P',
                  '--client',
                  '--include', 'common/client_marshallers.h',
                  '@INPUT0@', '@OUTPUT0@']

  client_marshallers = custom_target('client_marshallers',
                                     input : [spice_proto, client_demarshallers],
                                     output : ['generated_client_marshallers.c', 'generated_client_marshallers.h'],
                                     install : false,
                                     command : [codegen_cmd, codegen_args],
                                     depend_files : [spice_codegen_files, 'client_marshallers.h'])

  spice_common_client_sources = [
    client_demarshallers,
    client_marshallers,
    'client_marshallers.h',
    'ssl_verify.c',
    'ssl_verify.h',
  ]

  spice_common_client_lib = static_library('spice-common-client', spice_common_client_sources,
                                           install : false,
                                           dependencies : [spice_common_dep, gio2_deps])

  spice_common_client_dep = declare_dependency(sources : client_marshallers[1],
                                               link_with : spice_common_client_lib,
                                               dependencies : [spice_common_dep, gio2_deps])
endif

#
# libspice-common-server
#
if spice_common_generate_server_code
  structs_args = [
   '-M', 'String',
   '-M', 'Rect',
   '-M', 'Point',
   '-M', 'DisplayBase',
   '-M', 'Fill',
   '-M', 'Opaque',
   '-M', 'Copy',
   '-M', 'Blend',
   '-M', 'Blackness',
   '-M', 'Whiteness',
   '-M', 'Invers',
   '-M', 'Rop3',
   '-M', 'Stroke',
   '-M', 'Text',
   '-M', 'Transparent',
   '-M', 'AlphaBlend',
   '-M', 'Composite',
  ]

  targets = [
    { 'name' : 'server_demarshallers',
      'input' : [ spice_proto, client_demarshallers ],
      'output' : ['generated_server_demarshallers.c'],
      'codegen_args' : ['--generate-demarshallers',
                        '--server',
                        '--include', 'common/messages.h',
                        '@INPUT0@', '@OUTPUT0@'],
    },
    { 'name' : 'server_marshallers',
      'input' : [ spice_proto, client_demarshallers ],
      'output' : ['generated_server_marshallers.c', 'generated_server_marshallers.h'],
      'codegen_args' : ['--generate-marshallers',
                        '--generate-header',
                        '--server',
                        structs_args,
                        '--include', 'common/messages.h',
                        '@INPUT0@', '@OUTPUT0@']
    },
  ]

  spice_common_server_sources = []
  spice_common_server_dep_sources = []

  foreach t : targets
    target = custom_target(t['name'],
                           input : t['input'],
                           output : t['output'],
                           install : false,
                           command : [codegen_cmd, t['codegen_args']],
                           depend_files : [spice_codegen_files, 'messages.h'])
    spice_common_server_sources += target
    if t['output'].length() > 1
      spice_common_server_dep_sources += target[1]
    endif
  endforeach

  spice_common_server_lib = static_library('spice-common-server', spice_common_server_sources,
                                           install : false,
                                           dependencies : spice_common_dep)

  spice_common_server_dep = declare_dependency(sources : spice_common_server_dep_sources,
                                               link_with : spice_common_server_lib,
                                               dependencies : spice_common_dep)
endif
