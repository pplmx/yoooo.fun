---
title: expand IPv6 by shell
date: 2020-04-26T15:21:38+08:00
slug: ee868e029739ddae71e6a50a4c667d23
draft: false
lastmod: 2020-04-26T15:30:42+08:00
categories: [shell]
tags: [IPv6, shell]
keywords: expand, IPv6, shell 
description: expand IPv6
---
```bash
#!/usr/bin/env bash
function expand_ipv6() {
    # input: ::
    # output: 0:0:0:0:0:0:0:0
    # input: 2001:2d:1f::1
    # output: 2001:2d:1f:0:0:0:0:1
    local ipv6=${1}
    local colon_num=$(echo ${ipv6} | awk '{print gsub(/:/, "")}')
    local replace_str=""
    for (( i = 0; i <= $(( 7 - ${colon_num} )); ++ i )); do
        replace_str="${replace_str}:0"
    done
    replace_str="${replace_str}:"
    local ipv6_expanded=${ipv6/::/$replace_str}
    [[ ${ipv6_expanded} == *: ]] && ipv6_expanded="${ipv6_expanded}0"
    [[ ${ipv6_expanded} == :* ]] && ipv6_expanded="0${ipv6_expanded}"
    # return value
    echo ${ipv6_expanded}
}

function expand_expanded_ipv6() {
    # input: 0:0:0:0:0:0:0:0
    # output: 0000:0000:0000:0000:0000:0000:0000:0000
    # input: 2001:2d:1f:0:0:0:0:1
    # output: 2001:002d:001f:0000:0000:0000:0000:0001
    local expanded_ipv6=${1}
    local hex_arr=(${expanded_ipv6//:/ })
    for (( i = 0; i < 8; ++ i )); do
        local len=${#hex_arr[i]}
        for (( j = 4; j > ${len}; -- j )); do
            hex_arr[i]="0${hex_arr[i]}"
        done
    done
    # return value
    echo ${hex_arr[@]} | tr " " :
}

function convert_ipv6_to_decimal_basing_8bit() {
    # input(hexadecimal): 0000:0000:0000:0000:0000:0000:0000:0000
    # output(decimal): 00.00.00.00.00.00.00.00.00.00.00.00.00.00.00.00
    # input(hexadecimal): 2001:002d:001f:0000:0000:0000:0000:0001
    # output(decimal): 32.1.0.45.0.31.0.0.0.0.0.0.0.0.0.1
    local completed_ipv6=${1}
    local hex_arr=(${completed_ipv6//:/ })
    local hex_arr_split_by_8bit=()
    for (( i = 0; i < 8; ++ i )); do
        hex_arr_split_by_8bit=( ${hex_arr_split_by_8bit[@]} ${hex_arr[i]:0:2} ${hex_arr[i]:2} )
    done
    local dec_arr=()
    for (( i = 0; i < 16; ++ i )); do
        dec_arr=( ${dec_arr[@]} $(echo $(( 16#${hex_arr_split_by_8bit[i]} ))) )
    done
    # return value
    echo ${dec_arr[@]} | tr " " .
}

function convert_ipv6_to_decimal_basing_16bit() {
    # input(hexadecimal): 0:0:0:0:0:0:0:0
    # output(decimal): 0.0.0.0.0.0.0.0
    # input(hexadecimal): 2001:2d:1f:0:0:0:0:1
    # output(decimal): 8193.45.31.0.0.0.0.1
    local expanded_ipv6=${1}
    local hex_arr=(${expanded_ipv6//:/ })
    local dec_arr=()
    for (( i = 0; i < 8; ++ i )); do
        dec_arr=( ${dec_arr[@]} $(echo $(( 16#${hex_arr[i]} ))) )
    done
    # return value
    echo ${dec_arr[@]} | tr " " .
}

expand_ipv6 $1
expand_expanded_ipv6 $(expand_ipv6 $1)
convert_ipv6_to_decimal_basing_8bit $(expand_expanded_ipv6 $(expand_ipv6 $1))
convert_ipv6_to_decimal_basing_16bit $(expand_ipv6 $1)
```