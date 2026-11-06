# Match schemes

Companion to `README.md` for the fingerprint schemes module. This file specifies what a **match** scheme adds to the declarations, check records and results defined there. It adds nothing to the identity class and changes nothing in core. Section numbers continue the README's.

## 11. What a match scheme is

A match scheme derives a fingerprint from the subject's bytes and compares it against entries in a registry. It reports whether the subject meets a named decision policy for a named entry. The registry supplies the registrant's identity; the content carries nothing. "Content" here covers text, audio and image; this draft's fixtures are text.

## 12. What a match record shows

A `found` match record shows that an identified checker, applying the named decision policy, determined that the subject bytes match the named entry. That is the whole of the claim.

A match record does not show that the subject derives from the entry: a shared source, a quotation in a third text, or common phrasing produce the same result. It does not show that the content entered a model's context, was used in an output, or was reported completely. The registrant of an entry is not thereby the rights holder in it, and the entry's registration time is evidence of when a fingerprint was recorded, not of when or by whom the content was created. A `not_found` is not evidence that the subject was unused; §8 records the transformations each scheme does not survive.

A match record MUST NOT be presented as an identity record (§3). A consumer MAY combine a match record with identity, seeded-observation or entitlement evidence under its own policy; the combination belongs to the consumer, not to this module. Where a licence is in question, the check belongs to the entitlement module and a match record points to it rather than making it.

## 13. Declaration fields for match schemes

In addition to §4:

| Field | Meaning |
|---|---|
| `algorithm` | Normative reference for the fingerprint, with version, e.g. ISO 24138:2024 Content-Code Text v0 |
| `unit` | What one fingerprint covers: `document`, `block`, `segment`, or `window` with its parameters |
| `min_unit` | Smallest subject the scheme will check, as `{measure, value}` in the scheme's own measure (characters, tokens, seconds, bytes). Subjects below it return `indeterminate` with reason `below_min_unit` |
| `decision_policies` | Identifiers and digests of the policies the operator publishes (§14) |
| `registry` | Identifier of the registry the scheme matches against, and how an entry reference resolves |
| `false_positive_profile` | Plain statement of what unrelated content scores high under the scheme (boilerplate, standing paragraphs, very short units) |

`trust_anchors_supported` is omitted for match schemes. `time_anchors_supported` SHOULD include `registry_time`. A match scheme MAY declare `check_methods: reproducible` only for decision policies whose documents are public; `attested` is the path for undisclosed policies.

## 14. Decision policies

A decision policy is a digest-bound document the operator publishes. It states the scheme and version, the unit compared, the similarity measure and its direction, the threshold or rule that yields `found`, and how a subject is aligned to entries at `block` or `segment` unit (best-matching part, or all parts above threshold). A `reproducible` policy MUST state the threshold; an `attested` policy MAY omit it.

Changing any of these produces a new policy identifier. A record names the policy it applied, so a consumer always knows which rule produced the result without the record carrying scores.

## 15. Check record `signal` for match

With `found`, in addition to §5:

| Field | Meaning |
|---|---|
| `registry` | Registry identifier from the declaration |
| `entry` | Entry identifier within that registry |
| `entry_part` | For `block` or `segment` unit, which part of the entry matched; omitted at `document` unit. `subject.part_of` (§5) says which whole the subject belongs to; `entry_part` says which part of the entry it met |
| `policy` | `{id, digest}` of the decision policy applied |
| `registry_time` | `{anchor, at}`: the entry's registration time as the registry records it, with the anchor kind from `time_anchors_supported` |

`registry_time` is in the record so that a consumer can refuse a match whose registration post-dates the subject's event. A registry can be seeded with material the registrant does not control; this field is the consumer's lever against it.

## 16. Results for match schemes

- `found`: the subject meets the named decision policy for the named entry.
- `not_found`: no entry in the named registry meets the policy for the subject. This is a statement about the registry and policy, not about the subject's history.
- `indeterminate`, with `indeterminate_reason` one of `below_min_unit`, `part_under_document_resolution`, `registry_unresolvable`, `entry_unresolvable`, `policy_unresolvable`.
- `invalid` does not apply to match schemes, which have no signal to fail validation. A checker MUST use `indeterminate` where an identity check would use `invalid`.

## 17. Who checks, for match schemes

§7 applies. In particular, a registry operator that checks against its own registry is an owner-side checker: it declares the link as a `scheme_operator` relationship in the envelope (§6), and its records count as the operator's claims unless the check is `reproducible`. Because the operator sees every query and knows the registrant, a match checker SHOULD deliver each record to the content owner as well as to the emitter, whatever the common frame decides about append-only logs; delivery is cheap for this class and catches a withheld record at the moment it is made.

## 18. Reproducibility

A `reproducible` match check is one a consumer holding the subject bytes, the policy document and registry access can repeat and obtain the same result. The two independent checkers §9 requires MUST agree on every fixture row before a match scheme is marked stable. The fixtures are designed so that a second implementer needs only the algorithm reference, the policy document and the fixture manifest, not the first implementer's code.

## 19. Survivability for match schemes

The §8 matrix applies unchanged, with `observed_result` and `expected_result` taking `found` or `not_found`. Each match scheme ships rows for at least: identical; sub-edited; condensed rewrite; the same work updated with parts added and removed; an excerpt checked as a part; unrelated content on the same topic; and full paraphrase. The last two are deliberate `not_found` rows and the matrix says so.

The first two matrices are `fixtures/survivability-iscc-text.json` (ISO 24138 Content-Code Text, document and block unit) and `fixtures/survivability-winnowing-text.json` (winnowing k=25, w=4), seven rows each on synthetic CC0 fixtures, scores recomputed with iscc-core 1.3.0. Together they show why `unit` is declared rather than assumed: on the updated-version row the ISCC document-unit result is `not_found` while block-unit ISCC finds the excerpt and winnowing holds; and a Jaccard policy scores a part against a whole low regardless of origin, so parts need a containment policy, recorded as a known limit until one is specified.

## 20. Negative vectors for match schemes

In addition to §9: a `found` whose `registry_time` post-dates the subject's event time (`registry_time_after_event`); a `found` on a subject below `min_unit` (`below_min_unit`); a `found` naming a policy whose digest does not match the declaration (`policy_digest_mismatch`); a match record whose `signal` carries identity fields (already `match_presented_as_identity`); a `not_found` at `document` unit on a subject marked as a part (already `document_scheme_on_part`). The first two need the fixture events to carry a timestamp and the subject a `length`; both are proposed in the vector patch offered on 20 October.

## 21. Example record (abbreviated)

```json
{
  "scheme": "https://example.org/schemes/match-example",
  "declaration": {"ref": "https://example.org/schemes/match-example/declaration.json", "digest": "sha256:..."},
  "subject": {"role": "grounded_content", "digest": "sha256:...", "part_of": "sha256:..."},
  "event": {"session_id": "...", "event_id": "evt-chunk-a"},
  "result": "found",
  "signal": {
    "registry": "https://registry.example.org",
    "entry": "entry-42",
    "entry_part": "block:1",
    "policy": {"id": "https://registry.example.org/policy/1", "digest": "sha256:..."},
    "registry_time": {"anchor": "registry_time", "at": "2026-09-02T09:00:00Z"}
  },
  "method": "reproducible"
}
```

The envelope (§6) wraps this as for any record.

## 22. Open points for the match class

1. Containment policy for parts under local-overlap schemes (§19).
2. Audio and image schemes, and ISCC Data and Instance matrices for the disclosed-corpus case: follow once a second text checker exists.
3. Workflow level: declared once #6 defines V0–V3.
