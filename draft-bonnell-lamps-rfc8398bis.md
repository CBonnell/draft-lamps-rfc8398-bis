---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Internationalized Email Addresses in X.509 Certificates"
abbrev: "I18N Mail Addresses in X.509 Certificates"
category: std

docname: draft-bonnell-lamps-rfc8398bis-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: Security
# workgroup: WG Working Group
keyword:
 - EAI
 - PKIX
 - email address
updates: 5820
obsoletes: 8398
venue:
  group: "Limited Additional Mechanisms for PKIX and SMIME (lamps)"
  type: "Working Group"
  mail: "spasm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spasm/"
  github: "CBonnell/draft-lamps-rfc8398-bis"
  latest: "https://CBonnell.github.io/draft-lamps-rfc8398-bis/draft-bonnell-lamps-rfc8398bis.html"
author:
  -
    fullname: Alexey Melnikov
    organization: Isode Ltd
    street: 14 Castle Mews
    city: Hampton
    region: Middlesex
    code: TW12 2NP
    country: United Kingdom
    email: Alexey.Melnikov@isode.com
  -
    fullname: Wei Chuang
    organization: Google, Inc.
    street: 1600 Amphitheater Parkway
    city: Mountain View
    region: CA
    country: United States of America
    email: weihaw@google.com
  -
    fullname: Corey Bonnell
    organization: DigiCert
    city: Pittsburgh
    region: PA
    country: United States of America
    email: corey.bonnell@digicert.com

normative:
  RFC8399BIS:
    author:
      - ins: R. Housley
        fullname: Russ Housley
        org: Vigil Security, LLC
        city: Herndon
        region: VA
        country: United States of America
    target: https://datatracker.ietf.org/doc/draft-housley-lamps-rfc8399bis/
    title: Internationalization Updates to RFC 5280

informative:
  WEBER:
    author:
    - ins: C. Weber
      name: C. Weber
    date: March 2010
    target: https://www.lookout.net/files/Chris_Weber_Character%20Transformations%20v1.7_IUC33.pdf
    title: Attacking Software Globalization


--- abstract

This document defines a new name form for inclusion in the otherName
field of an X.509 Subject Alternative Name and Issuer Alternative
Name extension that allows a certificate subject to be associated
with an internationalized email address.

This document updates RFC 5280 and obsoletes RFC 8398.

--- middle

# Introduction

{{!RFC5280}} defines the rfc822Name subjectAltName name type for
representing email addresses as described in {{!RFC5321}}.  The syntax
of rfc822Name is restricted to a subset of US-ASCII characters and
thus can't be used to represent internationalized email addresses
{{!RFC6531}}.  This document defines a new otherName variant to
represent internationalized email addresses.  In addition this
document requires all email address domains in X.509 certificates to
conform to IDNA2008 {{!RFC5890}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Name Definitions

The GeneralName structure is defined in {{RFC5280}} and supports many
different name forms including otherName for extensibility.  This
section specifies the SmtpUTF8Mailbox name form of otherName so that
internationalized email addresses can appear in the subjectAltName of
a certificate, the issuerAltName of a certificate, or anywhere else
that GeneralName is used.

~~~
id-on-SmtpUTF8Mailbox OBJECT IDENTIFIER ::= { id-on 9 }

SmtpUTF8Mailbox ::= UTF8String (SIZE (1..MAX))
-- SmtpUTF8Mailbox conforms to Mailbox as specified
-- in Section 3.3 of RFC 6531. Additionally, all domain
-- labels included in the SmtpUTF8Mailbox value are
-- encoded as LDH-labels. In particular, domain labels
-- are not encoded as U-labels and instead are encoded
-- using their A-label representation.
~~~

When the subjectAltName (or issuerAltName) extension contains an
internationalized email address with a non-ASCII Local-part, the
address MUST be stored in the SmtpUTF8Mailbox name form of otherName.
The format of SmtpUTF8Mailbox is defined as the ABNF rule
SmtpUTF8Mailbox.  SmtpUTF8Mailbox is a modified version of the
internationalized Mailbox that was defined in Section 3.3 of
{{RFC6531}}, which was derived from Mailbox as defined in Section 4.1.2
of {{RFC5321}}.  {{RFC6531}} defines the following ABNF rules for Mailbox
whose parts are modified for internationalization: `Local-part`,
`Dot-string`, `Quoted-string`, `QcontentSMTP`, `Domain`, and `Atom`.
In particular, `Local-part` was updated to also support
UTF8-non-ascii.  UTF8-non-ascii was described by Section 3.1 of
{{!RFC6532}}. Also, domain was extended to support U-labels, as defined
in {{RFC5890}}.

This document further refines internationalized Mailbox ABNF rules as
described in {{RFC6531}} and calls this SmtpUTF8Mailbox.  In
SmtpUTF8Mailbox, labels that include non-ASCII characters MUST be
stored in A-label (rather than U-label) form {{RFC5890}}.  This
restriction reduces complexity for implementations of the certification
path validation algorithm defined in Section 6 of {{RFC5280}}.  In
SmtpUTF8Mailbox, domain labels that solely use ASCII characters (meaning
neither A- nor U-labels) SHALL use NR-LDH restrictions as specified by
Section 2.3.1 of {{RFC5890}}.  NR-LDH stands for "Non-Reserved Letters
Digits Hyphen" and is the set of LDH labels that do not have "--"
characters in the third and forth character position, which excludes
"tagged domain names" such as A-labels. To facilitate octet-for-octet
comparisons of SmtpUTF8Mailbox values, all NR-LDH and A-label labels
which constitute the domain part SHALL only be encoded with lowercase
letters. Consistent with the treatment of rfc822Name in {{RFC5280}},
SmtpUTF8Mailbox is an envelope `Mailbox` and has no phrase (such as a
common name) before it, has no comment (text surrounded in parentheses)
after it, and is not surrounded by "<" and ">" characters.

Due to name constraint compatibility reasons described in {{name-constraints}},
SmtpUTF8Mailbox subjectAltName MUST NOT be used unless the Local-part
of the email address contains non-ASCII characters.  When the Local-
part is ASCII, rfc822Name subjectAltName MUST be used instead of
SmtpUTF8Mailbox.  This is compatible with legacy software that
supports only rfc822Name (and not SmtpUTF8Mailbox).  The appropriate
usage of rfc822Name and SmtpUTF8Mailbox is summarized in Table 1
below.

SmtpUTF8Mailbox is encoded as UTF8String.  The UTF8String encoding
MUST NOT contain a Byte-Order-Mark (BOM) {{!RFC3629}} to aid consistency
across implementations, particularly for comparison.

| Local-part char | subjectAltName  |
|-----------------|-----------------|
|    ASCII-only   |    rfc822Name   |
|    non-ASCII    | SmtpUTF8Mailbox |
{: #santypes title="Email Address Formatting"}

Non-ASCII Local-part values may additionally include ASCII characters.

# IDNA2008

To facilitate comparison between email addresses, all email address
domains in X.509 certificates MUST conform to IDNA2008 {{RFC5890}} (and
avoid any "mappings" mentioned in that document).  Use of
non-conforming email address domains introduces the possibility of
conversion errors between alternate forms.  This applies to
SmtpUTF8Mailbox and rfc822Name in subjectAltName, issuerAltName, and
anywhere else that these are used.

# Matching of Internationalized Email Addresses in X.509 Certificates

Equivalence comparisons with SmtpUTF8Mailbox consist of
a domain part step and a Local-part step.  The comparison form for
Local-parts is always UTF-8.  The comparison form for domain parts
is always performed with the LDH-label ({{RFC5890}}) encoding of the
relevant domain labels. The comparison of LDH-labels in domain parts
reduces complexity for implementations of the certification path
validation algorithm as defined Section 6 of {{RFC5280}} by obviating
the need to convert domain labels to their Unicode representation.

Comparison of two SmtpUTF8Mailboxes is straightforward with no setup
work needed.  They are considered equivalent if there is an exact
octet-for-octet match.

Comparison of a SmtpUTF8Mailbox and rfc822Name will always fail.
SmtpUTF8Mailbox values SHALL contain a Local-part which includes
one or more non-ASCII characters, while rfc822Names only
include ASCII characters (including the Local-part). Thus, a
SmtpUTF8Mailbox and rfc822Name will never match.

Comparison of SmtpUTF8Mailbox values with internationalized email
addresses requires additional setup steps for domain part and
Local-part. The initial preparation for the email address to compare
with the SmtpUTF8Mailbox value is to remove any phrases, comments, and
"<" or ">" characters.

For the setup of the domain part, the following conversions SHALL be
performed:

1. Convert all labels which constitute the domain part that include
   non-ASCII characters to A-labels if not already in that form.
    a. Detect all U-labels present within the domain part using
       Section 5.1 of {{!RFC5891}}.
    b. Transform all detected U-labels (Unicode) to A-labels (ASCII) as
       specified in Section 5.5 of {{RFC5891}}.
2. Convert all uppercase letters found within the NR-LDH and A-label
   labels which constitute the domain part to lowercase letters.

For the setup of the Local-part, the Local-part MUST be verified to
conform to the requirements of {{!RFC6530}} and {{RFC6531}}, including
being a string in UTF-8 form.  In particular, the Local-
part MUST NOT be transformed in any way, such as by doing case
folding or normalization of any kind.  The `Local-part` part of an
internationalized email address is already in UTF-8. Once setup is
complete, they are again compared octet-for-octet.

To summarize non-normatively, the comparison steps, including setup,
are:

1.  If the domain contains U-labels, transform them to A-labels.
2.  If any NR-LDH or A-label domain label in the domain part
    contains uppercase letters, lowercase them.
3.  Compare strings octet-for-octet for equivalence.

This specification expressly does not define any wildcard characters,
and SmtpUTF8Mailbox comparison implementations MUST NOT interpret any
characters as wildcards.  Instead, to specify multiple email
addresses through SmtpUTF8Mailbox, the certificate MUST use multiple
subjectAltNames or issuerAltNames to explicitly carry any additional
email addresses.

# Name Constraints in Path Validation {#name-constraints}

This section updates Section 4.2.1.10 of {{RFC5280}} to extend
rfc822Name name constraints to SmtpUTF8Mailbox subjectAltNames.
SmtpUTF8Mailbox-aware path validators will apply name constraint
comparison to the subject distinguished name and both forms of
subject alternative names rfc822Name and SmtpUTF8Mailbox.

Both rfc822Name and SmtpUTF8Mailbox subject alternative names
represent the same underlying email address namespace.  Since legacy
CAs constrained to issue certificates for a specific set of domains
would lack corresponding UTF-8 constraints, {{RFC8399BIS}} updates,
modifies, and extends rfc822Name name constraints defined in
{{RFC5280}} to cover SmtpUTF8Mailbox subject alternative names.  This
ensures that the introduction of SmtpUTF8Mailbox does not violate
existing name constraints.  Since it is not valid to include
non-ASCII UTF-8 characters in the Local-part of rfc822Name name
constraints, and since name constraints that include a Local-part are
rarely, if at all, used in practice, name constraints updated in
{{RFC8399BIS}} allow the forms that represent all addresses at a host or
all mailboxes in a domain and deprecates rfc822Name name constraints
that represent a particular mailbox.  That is, rfc822Name constraints
with a Local-part SHOULD NOT be used.

Constraint comparison with SmtpUTF8Mailbox subjectAltName starts with
the setup steps defined by Section 5.  Setup converts the inputs of
the comparison (which is one of a subject distinguished name, an
rfc822Name, or an SmtpUTF8Mailbox subjectAltName, and one of an
rfc822Name name constraint) to constraint comparison form. For both the
name constraint and the subject, this will
 any domain NR-LDH labels.  Strip the Local-part and "@"
separator from each rfc822Name and SmtpUTF8Mailbox, leaving just the
domain part.  After setup, this follows the comparison steps defined
in Section 4.2.1.10 of {{RFC5280}} as follows.  If the resulting name
constraint domain starts with a "." character, then for the name
constraint to match, a suffix of the resulting subject alternative
name domain MUST match the name constraint (including the leading
".") octet-for-octet.  If the resulting name constraint domain does
not start with a "." character, then for the name constraint to
match, the entire resulting subject alternative name domain MUST
match the name constraint octet-for-octet.

Certificate Authorities that wish to issue CA certificates with email
address name constraints MUST use rfc822Name subject alternative
names only.  These MUST be IDNA2008-conformant names with no mappings
and with non-ASCII domains encoded in A-labels only.

The name constraint requirement with SmtpUTF8Mailbox subject
alternative name is illustrated in the non-normative diagram in
{{nctypes}}.  The first example (1) illustrates a permitted rfc822Name
ASCII-only host name name constraint and the corresponding valid
rfc822Name subjectAltName and SmtpUTF8Mailbox subjectAltName email
addresses.  The second example (2) illustrates a permitted rfc822Name
host name name constraint with A-label, and the corresponding valid
rfc822Name subjectAltName and SmtpUTF8Mailbox subjectAltName email
addresses.  Note that an email address with ASCII-only Local-part is
encoded as rfc822Name despite also having Unicode present in the
domain.

~~~
+-------------------------------------------------------------------+
|  Root CA Cert                                                     |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|  Intermediate CA Cert                                             |
|      Permitted                                                    |
|        rfc822Name: elementary.school.example.com (1)              |
|                                                                   |
|        rfc822Name: xn--pss25c.example.com (2)                     |
|                                                                   |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|  Entity Cert (w/explicitly permitted subjects)                    |
|    SubjectAltName Extension                                       |
|      rfc822Name: student@elemenary.school.example.com (1)         |
|      SmtpUTF8Mailbox: u+5B66u+751F@elementary.school.example.com  |
|        (1)                                                        |
|                                                                   |
|      rfc822Name: student@xn--pss25c.example.com (2)               |
|      SmtpUTF8Mailbox: u+533Bu+751F@xn--pss25c.example.com (2)     |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #nctypes title="Name Constraints with SmtpUTF8Name and rfc822Name"}

# Security Considerations

Use of SmtpUTF8Mailbox for certificate subjectAltName (and
issuerAltName) will incur many of the same security considerations as
in Section 8 in {{RFC5280}}, but it introduces a new issue by
permitting non-ASCII characters in the email address Local-part.
This issue, as mentioned in Section 4.4 of {{RFC5890}} and in Section 4
of {{RFC6532}}, is that use of Unicode introduces the risk of visually
similar and identical characters that can be exploited to deceive the
recipient.  The former document references some means to mitigate
against these attacks.  See {{WEBER}} for more background on security
issues with Unicode.

# Differences from RFC 8398

This document obsoletes {{!RFC8398}}. There are three major changes
defined in this specification which deviate from {{RFC8398}}:

1. In all cases, domain labels in mail addresses SHALL be encoded as
LDH-labels. In particular, domain names SHALL NOT be encoded using
U-Labels and instead use A-Labels.
2. To accommodate the first change listed above, the mail address
matching algorithm defined in Section 5 of {{RFC8398}} has been modified
to only accept domain labels that are encoded using their A-label
representation.
3. Additionally, the name constraints processing algorithm defined in
Section 6 of {{RFC8398}} has been modified to only accept domain labels
that are encoded using their A-label representation.

# IANA Considerations

Update the document reference for the SmtpUTF8Mailbox otherName in the
"SMI Security for PKIX Other Name Forms" (1.3.6.1.5.5.7.8) registry
from RFC 8398 to this document.

--- back

# ASN.1 Module

The following ASN.1 module normatively specifies the SmtpUTF8Mailbox
structure.  This specification uses the ASN.1 definitions from
{{?RFC5912}} with the 2002 ASN.1 notation used in that document.
{{RFC5912}} updates normative documents using older ASN.1 notation.

~~~
LAMPS-EaiAddresses-2016
{ iso(1) identified-organization(3) dod(6)
  internet(1) security(5) mechanisms(5) pkix(7) id-mod(0)
  id-mod-lamps-eai-addresses-2016(92) }

DEFINITIONS IMPLICIT TAGS ::=
BEGIN

IMPORTS
OTHER-NAME
FROM PKIX1Implicit-2009
  { iso(1) identified-organization(3) dod(6) internet(1) security(5)
  mechanisms(5) pkix(7) id-mod(0) id-mod-pkix1-implicit-02(59) }

id-pkix
FROM PKIX1Explicit-2009
  { iso(1) identified-organization(3) dod(6) internet(1) security(5)
  mechanisms(5) pkix(7) id-mod(0) id-mod-pkix1-explicit-02(51) } ;

--
-- otherName carries additional name types for subjectAltName,
-- issuerAltName, and other uses of GeneralNames.
--

id-on OBJECT IDENTIFIER ::= { id-pkix 8 }

SmtpUtf8OtherNames OTHER-NAME ::= { on-SmtpUTF8Mailbox, ... }

on-SmtpUTF8Mailbox OTHER-NAME ::= {
    SmtpUTF8Mailbox IDENTIFIED BY id-on-SmtpUTF8Mailbox
}

id-on-SmtpUTF8Mailbox OBJECT IDENTIFIER ::= { id-on 9 }

SmtpUTF8Mailbox ::= UTF8String (SIZE (1..MAX))
-- SmtpUTF8Mailbox conforms to Mailbox as specified
-- in Section 3.3 of RFC 6531. Additionally, all domain
-- labels included in the SmtpUTF8Mailbox value are
-- encoded as LDH-Labels. In particular, domain labels
-- are not encoded as U-Labels and instead are encoded
-- using their A-label representation.

END
~~~

# Example of SmtpUTF8Mailbox

This non-normative example demonstrates using SmtpUTF8Mailbox as an
otherName in GeneralName to encode the email address
"u+533Bu+751F@xn--pss25c.example.com".

The hexadecimal DER encoding of the block is:

~~~
a02b0608 2b060105 05070809 a01f0c1d e58cbbe7 949f4078 6e2d2d70
73733235 632e6578 616d706c 652e636f 6d
~~~

The text decoding is:

~~~
0  43: [0] {
2   8:   OBJECT IDENTIFIER '1 3 6 1 5 5 7 8 9'
12  31:   [0] {
14  29:     UTF8String 'u+533Bu+751F@xn--pss25c.example.com'
      :     }
      :   }
~~~

The example was encoded using Google's "der-ascii" program and the
above text decoding is an output of Peter Gutmann's "dumpasn1"
program.

# Acknowledgments
{:numbered="false"}

TODO

Previous document:

Thank you to Magnus Nystrom for motivating this document.  Thanks to
Russ Housley, Nicolas Lidzborski, Laetitia Baudoin, Ryan Sleevi, Sean
Leonard, Sean Turner, John Levine, and Patrik Falstrom for their
feedback.  Also special thanks to John Klensin for his valuable input
on internationalization, Unicode, and ABNF formatting; to Jim Schaad
for his help with the ASN.1 example and his helpful feedback; and
especially to Viktor Dukhovni for helping us with name constraints
and his many detailed document reviews.

This document:

David Benjamin
