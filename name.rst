.. _Name:

Name
----

An NDN Name is a hierarchical name for NDN content, which contains a sequence of name components.

NDN Name Format
~~~~~~~~~~~~~~~

We use a 2-level nested TLV to represent a name.
The Type in the outer TLV indicates this is a Name.
All inner TLVs have Type either NameComponent, NumberComponent, or ImplicitSha256DigestComponent, indicating that the name component is a generic name component, number, or implicit digest.
There is no restriction on the Value field in a name component, but empty value is not allowed:

::

    Name ::= NAME-TYPE TLV-LENGTH
               (NameComponent | NumberComponent | ImplicitSha256DigestComponent)*

    NameComponent ::= NAME-COMPONENT-TYPE TLV-LENGTH BYTE+

    NumberComponent ::= NUMBER-COMPONENT-TYPE TLV-LENGTH
                          nonNegativeInteger

    ImplicitSha256DigestComponent ::= IMPLICIT-SHA256-DIGEST-COMPONENT TLV-LENGTH(=32)
                                        BYTE{32}

.. % 0 or many name components in name
.. % 0 or many bytes in name component


``NameComponent`` is a generic name component, containing any application defined value.

``NumberComponent`` is a component that carries a non-negative integer.

``ImplicitSha256DigestComponent`` is an implicit SHA256 digest component.


NDN URI Scheme
~~~~~~~~~~~~~~

For textual representation, it is often convenient to use URI to represent NDN names.
Please refer to RFC 3986 (URI Generic Syntax) for background.

- The scheme identifier is ``ndn``.

- Component types have the following URI representations:

  * ``NameComponent``

    When producing a URI from an NDN Name, only the generic URI unreserved characters are left unescaped.
    These are the US-ASCII upper and lower case letters (A-Z, a-z), digits (0-9), and the four specials PLUS (+), PERIOD (.), UNDERSCORE (\_), and HYPHEN (-).
    All other characters are escaped using either the percent-encoding method of the URI Generic Syntax.

    To unambiguously represent name components that would collide with the use of . and .. for relative URIs, any component that consists solely of one or more periods is encoded using three additional periods.

  * ``NumberComponent``

    Number components start with ``number=`` prefix (case sensitive), following with textual representation of the number.
    For example, ``number=10`` represents a NumberComponent with encoded value of 10.
    The number must be decimal and should be without leading zeros.

  * ``ImplicitSha256DigestComponent``

    Implicit SHA256 digest component starts with ``sha256digest=`` prefix (case sensitive), following the digest represented as a sequence of 32 hexadecimal numbers.
    For example, ``sha256digest=893259d98aca58c451453f29ec7dc38688e690dd0b59ef4f3b9d33738bff0b8d``

- The authority component (the part after the initial ``//`` in the familiar http and ftp URI schemes) is not relevant to NDN.
  It should not be present, and it is ignored if it is present.

Implicit Digest Component
~~~~~~~~~~~~~~~~~~~~~~~~~

The Name of every piece of content includes as its final component a derived digest that ultimately makes the name unique.
This digest may occur in an Interest Name as an ``ImplicitSha256DigestComponent`` and MUST appear as the last component in the name.
This final component in the name is never included explicitly in the Data packet when it is transmitted on the wire.
It can be computed by any node based on the Data packet content.

The **implicit digest component** consists of the SHA-256 digest of the entire Data packet without the signature component.  Having this digest as the last name component enables us to achieve the following two goals:

- Identify one specific Data packet and no other.

- Exclude a specific Data packet in an Interest (independent from whether it has a valid signature).

Canonical Order
~~~~~~~~~~~~~~~

In several contexts in NDN packet processing, it is useful to have a consistent ordering of names and name components.

The order between individual name components is defined as follows:

- Any ``ImplicitSha256DigestComponent`` is less than any ``NumberComponent``, which is less than any ``NameComponent``

::

    ImplicitSha256DigestComponent  <  NumberComponent  <  NameComponent

- The order for the same type components is such that:

  * If *a* is shorter than *b* (i.e., has fewer bytes), then *a* comes before *b*.

  * If *a* and *b* have the same length, then they are compared in ASCII lexicographic order (e.g., ordering based on memcmp() operation.)

For Names, the ordering is just based on the ordering of the first component where they differ.
If one name is a proper prefix of the other, then it comes first.

..
       While the above defines generic order of any NDN names, :ref:`Interest selector <Selectors>` ``ChildSelector`` does not take into account the implicit digest componen, unless all other components are equal.


Changes from CCNx
~~~~~~~~~~~~~~~~~

- The name encoding is changed from binary XML to TLV format.

- The discussions on naming conventions and the use of special markers inside NameComponents are removed from packet specification, and will be covered by a separate technical document

.. (\cite{NamingConvention}).

- Deprecated zero-length name component.

- Number and implicit digest use dedicate TLV types
