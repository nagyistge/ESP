# Evident Security Platform (ESP) Custom Signatures Guide

## Introduction
Evident Security Platform (ESP) Signatures are the checks that run on top of the AWS API and Config data collected across your AWS accounts. There are two types of ESP Signatures, the default ones, built-in and maintained by Evident, and Custom Signatures maintained by you.  Custom signatures are enabled for ESP enterprise accounts and allow you to extend the functionality of ESP by enhancing, or even replacing, built-in signatures. 

The Custom Signatures DSL is an interpreted JavaScript language. The AWS-specific objects used follow the [AWS SDK for Ruby standards](http://docs.aws.amazon.com/sdkforruby/api/frames.html). ESP Custom signatures can be created here: esp.evident.io/control_panel/custom_signatures/new.

## Configuration Parameters
Every custom signature requires a `dsl.configure` function. It sets up parameters needed for the custom signature engine.

`c.module` is currently required to validate signatures and is used by built-in ESP signatures to define method names and classes dynamically during runtime. 
`c.identifier` is required for reports and being able to search for your specific custom signature. 
`c.description` is required for the descriptions on signatures within the metascrape console and to validate the signature.
`c.valid_regions` is not required. This is an array of regions that the signature should only run in. This allows you to isolate what regions to run the signature in. Ex [‘us_east_1’] meaning only run this signature in us-east-1.
`c.display_as` is not required. This was added for signatures when they are global, i.e. a signature is run in us-east-1, but the signature has no requirement on what region it runs in because the result is the same.
`c.deep_inspection` is not required. Deep inspection allows you to set extra data on the alert or alerts that the signature is returning. The Deep Inspection section below provides details about this parameter.
`c.unique_identifier` is not required. Unique identifier allows you to set data on a alert or alerts to make the signature more unique through data. This works in conjunction with deep inspection, and is explained more in the unique identifier section below.
