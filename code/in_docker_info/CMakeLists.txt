set(src
  docker_info.c
  docker_info_config.c)

FLB_PLUGIN(in_docker_info "${src}" "")
