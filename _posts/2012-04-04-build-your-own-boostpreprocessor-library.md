---
id: 592
title: Build your own boost::preprocessor library
date: 2012-04-04T11:29:27+00:00
author: nick
layout: post
permalink: /build-your-own-boostpreprocessor-library/
tags:
  - bcp
  - boost
  - consolidate
  - preprocessor
  - self contained
---
Recently I realized that using boost::preprocessor could improve a design in our project. We have a framework which allows consumers to listen to events emit by modules. But we need to write a wrapper for each event. for an event like OnLogin, we write the wrapper this way:

	void Fire_OnLogin(bool bReconnect)
	{
		BEGIN_FIRE_EVENT(IMyEvent)
			pEvent->OnLogin(bReconnect);
		END_FIRE_EVENT
	}
	
By using boost::pp, we could write something like the following to accomplish the same goal:

	MY_EVENT(IMyEvent, OnLogin, 1, (bool))
 
Here is how `MY_EVENT` is implemented:

	#include <boost/preprocessor/repetition.hpp>
	#include <boost/preprocessor/array.hpp>
	#define __MY_PARAMS(z, n, args) BOOST_PP_ARRAY_ELEM(n, args) p##n
	#define MY_EVENT(cls, evt, argCount, argList)	\
		void Fire_##evt(\
			 BOOST_PP_ENUM(argCount, __MY_PARAMS, (argCount, argList)))\
		{\
			BEGIN_FIRE_EVENT(cls);\
				pEvent->evt(BOOST_PP_ENUM_PARAMS(argCount, p));\
			END_FIRE_EVENT;\
		}
		
I'm not going to explain the code line by line. If you want to know more about `BOOST_PP_ENUM`, `BOOST_PP_ARRAY_EVEM`, .., please refer to the [boost documentation](http://www.boost.org/doc/libs/1_49_0/libs/preprocessor/doc/index.html)
Everything looks great, except that we don't have boost available in my project. And as a facility in a framework, I don't like the idea asking the customer to install boost before using it. As preprocessor is a header only library, maybe we could extract it using <a href="http://www.boost.org/doc/tools/bcp/index.html">bcp</a> and then ship it along with the framework. This sounds good, but let's see if we have better option. What if we build a preprocessor library? Note my wording, we're gonna *build* instead of writing one.
Thanks to g++ that it has a parameter `-M` for generating the include-dependency file list:

	nickx:/tmp $ cat boostpp.cpp
	#include <boost/preprocessor/config/config.hpp>
	#include <boost/preprocessor/repetition.hpp>
	#include <boost/preprocessor/array.hpp>
	nickx:/tmp $ g++ -M boostpp.cpp
	boostpp.o: boostpp.cpp \
	  /usr/local/include/boost/preprocessor/config/config.hpp \
	  /usr/local/include/boost/preprocessor/repetition.hpp \
	  /usr/local/include/boost/preprocessor/repetition/deduce_r.hpp \
	  /usr/local/include/boost/preprocessor/detail/auto_rec.hpp \
	  ...</pre>
	
cat these files to produce a single file: (Some bash tricks are applied to remove a few unwanted characters)

	nickx:/tmp $ cat $(g++ -M boostpp.cpp |grep boost/preprocessor/|sed -e 's/\\//g')
	# /* **************************************************************************
	#  *                                                                          *
	#  *     (C) Copyright Paul Mensonides 2002.
	#  *     Distributed under the Boost Software License, Version 1.0. (See
	#  *     accompanying file LICENSE_1_0.txt or copy at
	#  *     http://www.boost.org/LICENSE_1_0.txt)
	#  *                                                                          *
	#  ************************************************************************** */
	#
	# /* See http://www.boost.org for most recent version. */
	#
	# ifndef BOOST_PREPROCESSOR_CONFIG_CONFIG_HPP
	# define BOOST_PREPROCESSOR_CONFIG_CONFIG_HPP
	#
	# /* BOOST_PP_CONFIG_FLAGS */
	#
	# define BOOST_PP_CONFIG_STRICT() 0x0001
	# define BOOST_PP_CONFIG_IDEAL() 0x0002
	#
	# define BOOST_PP_CONFIG_MSVC() 0x0004
	# define BOOST_PP_CONFIG_MWCC() 0x0008
	# define BOOST_PP_CONFIG_BCC() 0x0010
	# define BOOST_PP_CONFIG_EDG() 0x0020
	...
	
for boost 1.48, the generated file has 8062 lines. don't forget removing the include directives.

	nickx:/tmp $ cat $(g++ -M boostpp.cpp |grep boost/preprocessor/|\
	sed -e 's/\\//g')|grep '# *include' -v|wc
	    7801   60707  584467
    
Almost done.
As we're still using the BOOST_PP_* macros, it would have name conflicts if a customer already has boost preprocessor included in her project. We could fix this simply by replacing &#8216;BOOST'.

	nickx:/tmp $ cat $(g++ -M boostpp.cpp |grep boost/preprocessor/|\
	sed -e 's/\\//g')|grep '# *include' -v|sed -e 's/BOOST/MY/g'>mypreprocessor.h

Here is a bash script to wrap all this up:

	#!/bin/bash 
	
	src=/tmp/boostpp.cpp
	
	cat>$src<<EOF
	#include <boost/preprocessor/config/config.hpp>
	#include <boost/preprocessor/repetition.hpp>
	#include <boost/preprocessor/array.hpp>
	EOF
	
	cat $(g++ -M $src |grep boost/preprocessor/|sed -e 's/\\//g')|grep include -v|sed -e 's/BOOST/MY/g'>mypreprocessor.h
	
Rewrite MY_EVENT as:

	#include "mypreprocessor.h"
	#define __MY_PARAMS(z, n, args) MY_PP_ARRAY_ELEM(n, args) p##n
	#define MY_EVENT(cls, evt, argCount, argList)	\
		void Fire_##evt(\
			 MY_PP_ENUM(argCount, __MY_PARAMS, (argCount, argList)))\
		{\
			BEGIN_FIRE_EVENT(cls);\
				pEvent->evt(MY_PP_ENUM_PARAMS(argCount, p));\
			END_FIRE_EVENT;\
		}
		
That's all. hope you enjoy it.
