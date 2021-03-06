# Based on original work by David Manura
# Copyright (C) 2007-2012 LuaDist.
# Copyright (C) 2013 Brian Sidebotham

# Redistribution and use of this file is allowed according to the terms of the
# MIT license.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.


include( CMakeParseArguments )

file( READ opensslconf.h.in CONF )
string( REPLACE "#undef OPENSSL_EXPORT_VAR_AS_FUNCTION" "#cmakedefine OPENSSL_EXPORT_VAR_AS_FUNCTION"
    CONF "${CONF}" )

set( CONF "
#define OPENSSL_NO_GMP
#define OPENSSL_NO_JPAKE
#define OPENSSL_NO_KRB5
#define OPENSSL_NO_MD2
#define OPENSSL_NO_RFC3779
#define OPENSSL_NO_STORE
#define OPENSSL_NO_DYNAMIC_ENGINE
#define OPENSSL_NO_SCTP
#define OPENSSL_NO_EC_NISTP_64_GCC_128
${CONF}" )
file( WRITE ${CMAKE_CURRENT_BINARY_DIR}/opensslconf.h.cmake "${CONF}" )

#~2DO: set BUILDINF_DATE to `date LC_ALL=C LC_TIME=C`
configure_file( buildinf.h.cmake buildinf.h )

if( WIN32 AND MINGW AND NOT CYGWIN )
  set( OPENSSL_EXPORT_VAR_AS_FUNCTION true )
endif()
configure_file( ${CMAKE_CURRENT_BINARY_DIR}/opensslconf.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/openssl/opensslconf.h )

set( LIBSRC cryptlib.c mem.c mem_clr.c mem_dbg.c cversion.c ex_data.c cpt_err.c
  ebcdic.c uid.c o_time.c o_str.c o_dir.c o_init.c )

include_directories( AFTER SYSTEM ${CMAKE_CURRENT_BINARY_DIR} .. ../include . asn1 evp modes )

if( BUILD_SHARED_LIBS )
  add_definitions( -DOPENSSL_BUILD_SHLIBCRYPTO )
endif()

macro( add_submodule dir )
    set( options )
    set( oneValueArgs )
    set( multiValueArgs EXHEADERS )
    cmake_parse_arguments( add_submodule "" "" "${multiValueArgs}" ${ARGN} )

    #message( STATUS "{dir} ${dir}" )
    #message( STATUS "{add_submodule_UNPARSED_ARGUMENTS} ${add_submodule_UNPARSED_ARGUMENTS}" )
    foreach( name ${add_submodule_UNPARSED_ARGUMENTS} )
        set( LIBSRC ${LIBSRC} ${dir}/${name} )
    endforeach( name )

    #message( STATUS "{add_submodule_EXHEADERS} ${add_submodule_EXHEADERS}" )
    foreach( name ${add_submodule_EXHEADERS} )
        set( EXHEADERS ${EXHEADERS} ${dir}/${name} )
    endforeach( name )

endmacro( add_submodule )

add_submodule( ../engines e_4758cca.c e_aep.c e_atalla.c e_cswift.c e_gmp.c e_chil.c
  e_nuron.c e_sureware.c e_ubsec.c e_padlock.c e_capi.c )
#~2DO: ok?
add_submodule( ../engines/ccgost e_gost_err.c gost2001.c gost2001_keyx.c gost89.c
  gost94_keyx.c gost_ameth.c gost_asn1.c gost_crypt.c gost_ctl.c gost_eng.c gosthash.c
  gost_keywrap.c gost_md.c gost_params.c gost_pmeth.c gost_sign.c )

# avoid conflict: ocsp.h and wincrypt.h
if( WIN32 AND NOT CYGWIN )
  set_property( SOURCE ../engines/e_capi.c APPEND PROPERTY COMPILE_DEFINITIONS HEADER_OCSP_H )
  #set_property( SOURCE ../engines/ccgost/gost_asn1.c APPEND PROPERTY COMPILE_DEFINITIONS
  # WIN32_LEAN_AND_MEAN )
endif( WIN32 AND NOT CYGWIN )

add_submodule( aes aes_core.c aes_misc.c aes_ecb.c aes_cbc.c aes_cfb.c aes_ofb.c
  aes_ctr.c aes_ige.c aes_wrap.c EXHEADERS aes.h )
add_submodule( asn1 a_object.c a_bitstr.c a_utctm.c a_gentm.c a_time.c a_int.c a_octet.c
  a_print.c a_type.c a_set.c a_dup.c a_d2i_fp.c a_i2d_fp.c a_enum.c a_utf8.c a_sign.c
  a_digest.c a_verify.c a_mbstr.c a_strex.c x_algor.c x_val.c x_pubkey.c x_sig.c
  x_req.c x_attrib.c x_bignum.c x_long.c x_name.c x_x509.c x_x509a.c x_crl.c x_info.c
  x_spki.c nsseq.c x_nx509.c d2i_pu.c d2i_pr.c i2d_pu.c i2d_pr.c t_req.c t_x509.c
  t_x509a.c t_crl.c t_pkey.c t_spki.c t_bitst.c tasn_new.c tasn_fre.c tasn_enc.c
  tasn_dec.c tasn_utl.c tasn_typ.c tasn_prn.c ameth_lib.c f_int.c f_string.c n_pkey.c
  f_enum.c x_pkey.c a_bool.c x_exten.c bio_asn1.c bio_ndef.c asn_mime.c asn1_gen.c
  asn1_par.c asn1_lib.c asn1_err.c a_bytes.c a_strnid.c evp_asn1.c asn_pack.c p5_pbe.c
  p5_pbev2.c p8_pkey.c asn_moid.c EXHEADERS asn1.h asn1_mac.h asn1t.h )
add_submodule( bf bf_skey.c bf_ecb.c bf_enc.c bf_cfb64.c bf_ofb64.c EXHEADERS blowfish.h )
add_submodule( bio bio_lib.c bio_cb.c bio_err.c bss_mem.c bss_null.c bss_fd.c bss_file.c
  bss_sock.c bss_conn.c bf_null.c bf_buff.c b_print.c b_dump.c b_sock.c bss_acpt.c
  bf_nbio.c bss_log.c bss_bio.c bss_dgram.c # bf_lbuf.c
  EXHEADERS bio.h )
add_submodule( bn bn_add.c bn_div.c bn_exp.c bn_lib.c bn_ctx.c bn_mul.c bn_mod.c
  bn_print.c bn_rand.c bn_shift.c bn_word.c bn_blind.c bn_kron.c bn_sqrt.c bn_gcd.c
  bn_prime.c bn_err.c bn_sqr.c bn_asm.c bn_recp.c bn_mont.c bn_mpi.c bn_exp2.c bn_gf2m.c
  bn_nist.c bn_depr.c bn_const.c EXHEADERS bn.h )
add_submodule( buffer buffer.c buf_err.c buf_str.c EXHEADERS buffer.h )
add_submodule( camellia camellia.c cmll_misc.c cmll_ecb.c cmll_cbc.c cmll_ofb.c
  cmll_cfb.c cmll_ctr.c cmll_utl.c EXHEADERS camellia.h )
add_submodule( cast c_skey.c c_ecb.c c_enc.c c_cfb64.c c_ofb64.c EXHEADERS cast.h )
add_submodule( cmac cm_ameth.c cm_pmeth.c cmac.c EXHEADERS cmac.h )
add_submodule( cms cms_lib.c cms_asn1.c cms_att.c cms_io.c cms_smime.c cms_err.c
  cms_sd.c cms_dd.c cms_cd.c cms_env.c cms_enc.c cms_ess.c cms_pwri.c EXHEADERS cms.h )
add_submodule( comp comp_lib.c comp_err.c c_rle.c c_zlib.c EXHEADERS comp.h )
add_submodule ( conf conf_err.c conf_lib.c conf_api.c conf_def.c conf_mod.c conf_mall.c
  conf_sap.c EXHEADERS conf.h conf_api.h )
add_submodule( des cbc_cksm.c cbc_enc.c cfb64enc.c cfb_enc.c ecb3_enc.c ecb_enc.c
  enc_read.c enc_writ.c fcrypt.c ofb64enc.c ofb_enc.c pcbc_enc.c qud_cksm.c rand_key.c
  rpc_enc.c set_key.c des_enc.c fcrypt_b.c xcbc_enc.c str2key.c cfb64ede.c ofb64ede.c
  ede_cbcm_enc.c des_old.c des_old2.c read2pwd.c EXHEADERS des.h des_old.h )
add_submodule( dh dh_asn1.c dh_gen.c dh_key.c dh_lib.c dh_check.c dh_err.c dh_depr.c
  dh_ameth.c dh_pmeth.c dh_prn.c EXHEADERS dh.h )
add_submodule( dsa dsa_gen.c dsa_key.c dsa_lib.c dsa_asn1.c dsa_vrf.c dsa_sign.c
  dsa_err.c dsa_ossl.c dsa_depr.c dsa_ameth.c dsa_pmeth.c dsa_prn.c EXHEADERS dsa.h )
add_submodule( dso dso_dl.c dso_dlfcn.c dso_err.c dso_lib.c dso_null.c dso_openssl.c
  dso_win32.c dso_vms.c dso_beos.c EXHEADERS dso.h )
add_submodule( ec ec_lib.c ecp_smpl.c ecp_mont.c ecp_nist.c ec_cvt.c ec_mult.c ec_err.c
  ec_curve.c ec_check.c ec_print.c ec_asn1.c ec_key.c ec2_smpl.c ec2_mult.c ec_ameth.c
  ec_oct.c ec_pmeth.c eck_prn.c ecp_oct.c ec2_oct.c EXHEADERS ec.h )
add_submodule( ecdh ech_lib.c ech_ossl.c ech_key.c ech_err.c EXHEADERS ecdh.h )
add_submodule( ecdsa ecs_lib.c ecs_asn1.c ecs_ossl.c ecs_sign.c ecs_vrf.c ecs_err.c
  EXHEADERS ecdsa.h )
add_submodule( engine eng_err.c eng_lib.c eng_list.c eng_init.c eng_ctrl.c eng_table.c
  eng_pkey.c eng_fat.c eng_all.c tb_rsa.c tb_dsa.c tb_ecdsa.c tb_dh.c tb_ecdh.c tb_rand.c
  tb_store.c tb_cipher.c tb_digest.c tb_pkmeth.c tb_asnmth.c eng_openssl.c eng_cnf.c
  eng_dyn.c eng_cryptodev.c eng_rdrand.c eng_rsax.c EXHEADERS engine.h )
add_submodule( err err.c err_all.c err_prn.c EXHEADERS err.h )
add_submodule( evp encode.c digest.c evp_enc.c evp_key.c evp_acnf.c e_des.c e_bf.c
  e_idea.c e_des3.c e_camellia.c e_rc4.c e_aes.c names.c e_seed.c e_xcbc_d.c e_rc2.c
  e_cast.c e_rc5.c m_null.c m_md2.c m_md4.c m_md5.c m_sha.c m_sha1.c m_wp.c m_dss.c
  m_dss1.c m_mdc2.c m_ripemd.c m_ecdsa.c p_open.c p_seal.c p_sign.c p_verify.c p_lib.c
  p_enc.c p_dec.c bio_md.c bio_b64.c bio_enc.c evp_err.c e_null.c c_all.c c_allc.c
  c_alld.c evp_lib.c bio_ok.c evp_pkey.c evp_pbe.c p5_crpt.c p5_crpt2.c e_old.c pmeth_lib.c
  pmeth_fn.c pmeth_gn.c m_sigver.c evp_cnf.c e_rc4_hmac_md5.c e_aes_cbc_hmac_sha1.c
  EXHEADERS evp.h )
add_submodule( hmac hmac.c hm_ameth.c hm_pmeth.c EXHEADERS hmac.h )
add_submodule( idea i_cbc.c i_cfb64.c i_ofb64.c i_ecb.c i_skey.c EXHEADERS idea.h )
# add_submodule(jpake jpake.c jpake_err.c EXHEADERS jpake.h)
add_submodule( krb5 krb5_asn.c EXHEADERS krb5_asn.h )
add_submodule( lhash lhash.c lh_stats.c EXHEADERS lhash.h )
# add_submodule(md2 md2_dgst.c md2_one.c EXHEADERS md2.h )
add_submodule( md4 md4_dgst.c md4_one.c EXHEADERS md4.h )
add_submodule( md5 md5_dgst.c md5_one.c EXHEADERS md5.h )
add_submodule( mdc2 mdc2dgst.c mdc2_one.c EXHEADERS mdc2.h )
add_submodule( modes cbc128.c ccm128.c ctr128.c cts128.c cfb128.c gcm128.c ofb128.c
  xts128.c EXHEADERS modes.h )
add_submodule( objects o_names.c obj_dat.c obj_lib.c obj_err.c obj_xref.c EXHEADERS
  objects.h obj_mac.h )
add_submodule( ocsp ocsp_asn.c ocsp_ext.c ocsp_ht.c ocsp_lib.c ocsp_cl.c ocsp_srv.c
  ocsp_prn.c ocsp_vfy.c ocsp_err.c EXHEADERS ocsp.h )
add_submodule( pem pem_sign.c pem_seal.c pem_info.c pem_lib.c pem_all.c pem_err.c
  pem_x509.c pem_xaux.c pem_oth.c pem_pk8.c pem_pkey.c pvkfmt.c EXHEADERS pem.h pem2.h )
add_submodule( pkcs12 p12_add.c p12_asn.c p12_attr.c p12_crpt.c p12_crt.c p12_decr.c
  p12_init.c p12_key.c p12_kiss.c p12_mutl.c p12_utl.c p12_npas.c pk12err.c p12_p8d.c
  p12_p8e.c EXHEADERS pkcs12.h )
add_submodule( pkcs7 pk7_asn1.c pk7_lib.c pkcs7err.c pk7_doit.c pk7_smime.c pk7_attr.c
  pk7_mime.c bio_pk7.c EXHEADERS pkcs7.h )
add_submodule( pqueue pqueue.c EXHEADERS pqueue.h )
add_submodule( rand md_rand.c randfile.c rand_lib.c rand_err.c rand_egd.c rand_win.c
  rand_unix.c rand_os2.c rand_nw.c EXHEADERS rand.h )
add_submodule( rc2 rc2_ecb.c rc2_skey.c rc2_cbc.c rc2cfb64.c rc2ofb64.c EXHEADERS
  rc2.h )
add_submodule( rc4 rc4_skey.c rc4_enc.c rc4_utl.c EXHEADERS rc4.h )
add_submodule( rc5 rc5_skey.c rc5_ecb.c rc5_enc.c rc5cfb64.c rc5ofb64.c
  EXHEADERS rc5.h )
add_submodule( ripemd rmd_dgst.c rmd_one.c EXHEADERS ripemd.h )
add_submodule( rsa rsa_eay.c rsa_gen.c rsa_lib.c rsa_sign.c rsa_saos.c rsa_err.c
  rsa_pk1.c rsa_ssl.c rsa_none.c rsa_oaep.c rsa_chk.c rsa_null.c rsa_pss.c rsa_x931.c
  rsa_asn1.c rsa_depr.c rsa_ameth.c rsa_prn.c rsa_pmeth.c rsa_crpt.c EXHEADERS rsa.h )
add_submodule ( seed seed.c seed_ecb.c seed_cbc.c seed_cfb.c seed_ofb.c EXHEADERS
  seed.h )
add_submodule( sha sha_dgst.c sha1dgst.c sha_one.c sha1_one.c sha256.c sha512.c
  EXHEADERS sha.h )
add_submodule( srp srp_lib.c srp_vfy.c EXHEADERS srp.h srp_grps.h srp_lcl.h )
add_submodule( stack stack.c EXHEADERS stack.h safestack.h )
#add_submodule(store str_err.c str_lib.c str_meth.c str_mem.c
#   EXHEADERS store.h )
add_submodule( ts ts_err.c ts_req_utils.c ts_req_print.c ts_rsp_utils.c ts_rsp_print.c
  ts_rsp_sign.c ts_rsp_verify.c ts_verify_ctx.c ts_lib.c ts_conf.c ts_asn1.c EXHEADERS
  ts.h )
add_submodule( txt_db txt_db.c EXHEADERS txt_db.h )
add_submodule( ui ui_err.c ui_lib.c ui_openssl.c ui_util.c ui_compat.c EXHEADERS
  ui.h ui_compat.h )
add_submodule( whrlpool wp_dgst.c wp_block.c EXHEADERS whrlpool.h )
add_submodule( x509 x509_def.c x509_d2.c x509_r2x.c x509_cmp.c x509_obj.c x509_req.c
  x509spki.c x509_vfy.c x509_set.c x509cset.c x509rset.c x509_err.c x509name.c x509_v3.c
  x509_ext.c x509_att.c x509type.c x509_lu.c x_all.c x509_txt.c x509_trs.c by_file.c
  by_dir.c x509_vpm.c EXHEADERS x509.h x509_vfy.h )
add_submodule( x509v3 v3_bcons.c v3_bitst.c v3_conf.c v3_extku.c v3_ia5.c v3_lib.c
  v3_prn.c v3_utl.c v3err.c v3_genn.c v3_alt.c v3_skey.c v3_akey.c v3_pku.c v3_int.c
  v3_enum.c v3_sxnet.c v3_cpols.c v3_crld.c v3_purp.c v3_info.c v3_ocsp.c v3_akeya.c
  v3_pmaps.c v3_pcons.c v3_ncons.c v3_pcia.c v3_pci.c pcy_cache.c pcy_node.c pcy_data.c
  pcy_map.c pcy_tree.c pcy_lib.c v3_asid.c v3_addr.c EXHEADERS x509v3.h )
# dirs not included: threads

add_submodule( . EXHEADERS crypto.h opensslv.h opensslconf.h ebcdic.h symhacks.h
  ossl_typ.h )

if(BUILD_OBJECT_LIBRARY_ONLY)
  if( WIN32 AND MINGW AND NOT CYGWIN )
	add_definitions(-DOPENSSL_THREADS -DDSO_WIN32 -DWIN32_LEAN_AND_MEAN)
  endif( WIN32 AND MINGW AND NOT CYGWIN )
  add_library( crypto OBJECT ${LIBSRC} ${OBJECTS_SRC} )
  set( crypto_headers ${EXHEADERS} )
  list( REMOVE_ITEM crypto_headers ./opensslconf.h )
  file( COPY ${crypto_headers} DESTINATION openssl )
else()
  add_library( crypto ${LIBSRC} ${OBJECTS_SRC} )

  if( WIN32 AND NOT CYGWIN )
    target_link_libraries( crypto ws2_32 crypt32 )
  endif()

  install( DIRECTORY ${PROJECT_SOURCE_DIR}/include
      DESTINATION ./ )

  install( TARGETS crypto
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      ARCHIVE DESTINATION lib )
endif()
