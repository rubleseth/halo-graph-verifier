# Graph claim verification

## Language

**Graph claim** is a proposed relationship or attribute with a specific subject, scope and value.

**Decision-time evidence packet** is the source material and context the verifier could access when judging one claim. A later lookup creates a new packet.

**Reviewed verdict** is an independently checked judgment of what the packet establishes about a claim. A confidence score measures a model output and can be compared against reviewed verdicts.

**Single-call adaptive reasoning** is a model choosing how much reasoning to spend within one generation before returning a verdict.

**Paired replay** is an offline comparison of direct and thinking modes on the same frozen checkpoint and input packet.

**Unsupported acceptance** is a claim accepted as supported when the reviewed packet contradicts it or leaves evidence insufficient.
