# Open Tasks and Questions

- Make spec use RFC 2119
- What file extensions to use?
- Is there a standard MessagePack file extension?
- Is it worth creating new MIME types?
- Can JSON-LD be incorporated in a *simple* way?
- Can JSON-LD offer a more robust way for implementing extensions?
- Implementations in RDKit and Open Babel (JSON-cpp dependency?)
- A test suite for validating implementations (giving % conformance to specification)
- String encoding?
- null value vs missing fields. Does a null value ever have a different meaning to a missing field? If a missing field has an implied default value?
- Timestamps - Strings (RFC 3339?)
- Handling int vs. float vs. decimal with just a JSON number type
- 'JSON lines' format? Multiple root-level objects?
- Does no isotope info mean most abundant isotope, or the naturally occurring ratio?
- Exotic stereochemistry. Atropisomerism.
- Allow extension not to use the `x-` prefix under certain circumstances?
    + e.g. only for a single field in the top-level container that is the extension name?
    + e.g. only when it has a specification that has been reviewed and merged into this repository?
- `hcount` vs. `hydrogens`?
- It may not be a great idea to have extension specifcations actually in this repository. It makes it more complicated to use the git history and tags to access older versions of specifications. Maybe encourage a separate repo for each, and link to them from here once they have been reviewed/approved.
- Query features
- Conformer fields:
    + `units`: `angstroms`, `pixels`, etc.
    + `type`: `experimental`, `computed`, etc.
- Bond `style` vs. `type`. It is probably better to store bond type as `hydrogen`, `complex`, `dative` rather than depiction style `dashed`, `dotted`, `arrow`, etc.
- Lone pair in tetrahedral stereo?
