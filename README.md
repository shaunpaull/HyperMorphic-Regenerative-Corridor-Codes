# HyperMorphic-Regenerative-Corridor-Codes
HyperMorphic Regenerative Corridor Codes: self-healing erasure coding where valid shard corridors regenerate missing data and migrate across changing geometries.


HyperMorphic Regenerative Corridor Codes

Self-healing erasure coding by corridor recovery and geometry regeneration.

HyperMorphic Regenerative Corridor Codes, or HRC, extend HyperMorphic Corridor Codes by adding repair, regeneration, and geometry migration.

Where HCC asks:

Do the surviving shards form a valid reconstruction corridor?

HRC asks:

Can a surviving corridor regenerate missing or corrupted shards, then optionally migrate the codeword into a new geometry while preserving the original data invariant?

⸻

Core Idea

Traditional erasure coding usually follows this pattern:

encode → lose shards → decode original → re-encode replacements

HRC turns this into a regenerative lifecycle:

encode
  → damage / erasure / corruption
  → corridor-admissible decode
  → regenerate missing shards
  → verify invariant preservation
  → optionally migrate into a new geometry

The codeword is no longer treated as a fixed static object. It becomes a living codeword that can survive damage, repair itself, and reconfigure its representation geometry across generations.

⸻

One-Line Summary

HRC is a prototype self-healing erasure code where valid shard corridors regenerate missing data and migrate across changing geometries.

⸻

What Makes HRC Different?

Fixed threshold codes ask:

Do I have at least k shards?

HyperMorphic Corridor Codes ask:

Do the surviving shards span a valid reconstruction corridor?

HyperMorphic Regenerative Corridor Codes ask:

Can that corridor regenerate the codeword and preserve continuity across generations?

That gives HRC three layers:

1. Corridor recovery
    Data is reconstructed only when surviving shards satisfy an algebraic-geometric corridor predicate.
2. Regenerative repair
    Missing or corrupted shards are regenerated from the surviving corridor.
3. Geometry migration
    A repaired codeword can be re-encoded into a different shard geometry, such as changing n, lane count, or threshold range.

⸻

Primitive Definition

An HRC system is parameterized as:

HRC(n, D, W, E_levels, k_low, k_high)

Where:

Parameter	Meaning
n	total number of shards
D	number of representation lanes / geometry axes
W	causal entropy window size
E_levels	number of entropy quantization levels
k_low	minimum shard count in low-complexity states
k_high	minimum shard count in high-complexity states

Each shard belongs to a representation lane. Recovery depends not only on how many shards survive, but on whether they span enough lanes and algebraic capacity for the current state.

⸻

Encoding

At each chunk position:

H(pos)     = entropy of previous W decoded chunks
e(pos)     = quantized entropy state
p_i(e)     = adaptive prime modulus for shard i
b_i(e)     = SafeGear-style base with gcd(b_i, p_i) = 1
λ_i(pos,e) = SHAKE128-derived position/state mixing term

Encoding uses:

r_i(pos) = (b_i(e) · C(pos) + λ_i(pos,e)) mod p_i(e)

Where:

* C(pos) is the chunk integer
* r_i(pos) is shard i’s residue
* p_i(e) is the state-dependent modulus
* b_i(e) is the coprime modular base
* λ_i(pos,e) is the position/state mixing term

⸻

Decoding

For a selected surviving corridor S:

c_i(pos) = (r_i(pos) - λ_i(pos,e)) · b_i(e)^(-1) mod p_i(e)
C(pos) = CRT({c_i mod p_i(e)} for i in S)

The selected subset must satisfy the corridor predicate before reconstruction is accepted.

⸻

Corridor Predicate

A shard subset S is accepted only if:

|S|                  ≥ required_shards(e)
lane_count(S)        ≥ required_lanes(e)
∏ p_i(e)             > chunk_space
corridor_score(S,e)  ≥ required_score(e)

In other words:

recoverable ≠ merely enough shards
recoverable = enough invariant span

This allows two subsets with the same shard count to behave differently.

A full-span subset may recover.
A clustered subset with the same number of shards may be rejected.

⸻

Regenerative Repair

HRC adds a repair operator:

Repair(Surviving Shards) → Repaired Codeword

Repair works by:

1. validating surviving shards,
2. finding a valid reconstruction corridor,
3. decoding the original payload,
4. regenerating missing or corrupted shards,
5. assigning a new generation number,
6. emitting a repair certificate.

Example lifecycle:

Generation 0: encode 12 shards
Damage: erase 4 shards
Corridor: valid
Repair: regenerate erased shards
Generation 1: full codeword restored
Hash preserved: true

⸻

Geometry Migration

HRC can also migrate a codeword into a new geometry:

HRC(n=12, D=4) → HRC(n=16, D=8)

This means the payload can survive while the coding structure changes.

Migration works by:

1. decoding through the old surviving corridor,
2. verifying the data invariant,
3. constructing a new HRC codec with different geometry,
4. re-encoding into the new geometry,
5. producing a migration certificate.

Example:

old geometry: HRC(n=12,D=4,W=8,E=9,low=4,high=6)
new geometry: HRC(n=16,D=8,W=8,E=9,low=5,high=8)
data hash preserved: true

This is the central HyperMorphic idea:

information survives transformation.

⸻

Features

* Corridor-admissible recovery
    * Recovery requires invariant span, not just shard count.
* Self-healing repair
    * Missing shards can be regenerated from a valid surviving corridor.
* Corruption handling
    * HMAC validation identifies invalid/tampered shards in the prototype setting.
* Geometry migration
    * Codewords can be re-encoded into a new shard/lane geometry.
* Generation tracking
    * Shards carry generation metadata and parent fingerprints.
* Repair certificates
    * Repair outputs include regenerated indices, valid shard counts, data hash, corridor score, and capacity margin.
* Pure Python prototype
    * No external dependencies required.

⸻

Example Test Results

The included Colab/demo test suite validates:

14/14 tests passed
ALL PASS — HRC primitive validated

Test coverage includes:

* basic encode/decode
* same-geometry repair
* erased shard regeneration
* corruption repair
* geometry migration
* invalid corridor rejection
* repeated repair cycles
* performance benchmark

⸻

Quick Start

Clone the repository:

git clone https://github.com/YOUR_USERNAME/hypermorphic-regenerative-corridor-codes.git
cd hypermorphic-regenerative-corridor-codes

Run the demo:

python hrc.py

Or paste the full code into Google Colab and run the cell.

⸻

Minimal Usage

from hrc import HyperMorphicRegenerativeCorridorCode
hrc = HyperMorphicRegenerativeCorridorCode(
    n=12,
    geometry_dim=4,
    W=8,
    E_levels=9,
    low_shards=4,
    high_shards=6,
)
data = b"HyperMorphic Regenerative Corridor Codes"
shards = hrc.encode(data)
recovered = hrc.decode(shards)
assert recovered == data

⸻

Repair Example

data = b"self-healing payload" * 100
hrc = HyperMorphicRegenerativeCorridorCode()
shards = hrc.encode(data)
damaged, erased = hrc.damage_random(shards, count=4, seed=42)
repaired, repair_cert = hrc.repair(damaged)
assert hrc.decode(repaired) == data
print(repair_cert)

Example certificate fields:

{
    "mode": "same_geometry_repair",
    "old_geometry": "...",
    "new_geometry": "...",
    "generation_new": 1,
    "kept_indices": [...],
    "regenerated_indices": [...],
    "valid_before": 8,
    "valid_after": 12,
    "data_hash": "...",
    "data_hash_preserved": True
}

⸻

Geometry Migration Example

data = b"geometry migration payload" * 100
old_codec = HyperMorphicRegenerativeCorridorCode(
    n=12,
    geometry_dim=4,
    low_shards=4,
    high_shards=6,
)
old_shards = old_codec.encode(data)
damaged, erased = old_codec.damage_random(old_shards, count=3, seed=7)
new_codec, new_shards, migration_cert = old_codec.repair_into_new_geometry(
    damaged,
    new_n=16,
    new_D=8,
    new_low_shards=5,
    new_high_shards=8,
)
assert new_codec.decode(new_shards) == data
print(migration_cert)

⸻

Same Count, Different Geometry

HRC inherits the key HCC property:

good_ids = [0, 1, 2, 3, 4, 5]      # 6 shards, full lane span
bad_ids  = [0, 1, 4, 5, 8, 9]      # 6 shards, clustered lanes

Both subsets have 6 shards.

But the full-span subset may be accepted, while the clustered subset may be rejected.

This demonstrates that HRC is not reducible to a fixed scalar threshold.

⸻

Comparison

Property	Fixed Threshold Code	HCC	HRC
Fixed k-of-n recovery	yes	no	no
State-dependent threshold	no	yes	yes
Geometry-aware recovery	no	yes	yes
Same-count subsets can differ	no	yes	yes
Corridor certificate	no	yes	yes
Regenerates missing shards	usually via re-encode	no	yes
Repairs corrupted shards	external	prototype	prototype
Migrates into new geometry	no	no	yes
Generation continuity metadata	no	no	yes

⸻

Formal Claim

A careful version of the HRC claim is:

HyperMorphic Regenerative Corridor Codes extend corridor-admissible erasure coding with a repair operator that reconstructs the payload from a valid surviving corridor, regenerates missing or invalid shards, and optionally migrates the codeword into a new representation geometry while preserving a verifiable data invariant.

⸻

What This Is Not Claiming

This prototype does not claim:

* production readiness
* cryptographic secrecy
* replacement of Reed–Solomon
* formal Byzantine security
* information-theoretic superiority
* a proven new MDS code family
* optimized storage efficiency

It is a research prototype for exploring:

* state-dependent recovery
* geometry-aware erasure coding
* self-healing codewords
* adaptive shard topologies
* HyperMorphic representation survival

⸻

Roadmap

Planned next steps:

* add Reed–Solomon baseline
* add fixed CRT baseline
* add HTC and HCC benchmark comparison
* add clustered lane failure benchmarks
* add adversarial damage policies
* add repair-cost metrics
* add serialization/deserialization
* add file-based demos
* add visual corridor graphs
* add proof section
* add HoloRAID integration
* add optimized CRT implementation
* add Colab notebook with plots

⸻

Suggested Paper Title

HyperMorphic Regenerative Corridor Codes: Self-Healing Erasure Coding by Geometry-Preserving Repair

⸻

Suggested Citation

@misc{hypermorphic_regenerative_corridor_codes,
  title  = {HyperMorphic Regenerative Corridor Codes: Self-Healing Erasure Coding by Geometry-Preserving Repair},
  author = {Gerrard, Shaun Paul},
  year   = {2026},
  note   = {Research prototype}
}

⸻

License

Choose a license before release.

Recommended options:

* MIT for maximum openness
* Apache-2.0 for permissive use with patent-related clarity
* AGPL-3.0 if you want network-use modifications to remain open

⸻

Bottom Line

HRC turns a codeword into a regenerative object:

damage → corridor decode → repair → verify → migrate

The central idea is simple:

the data survives because enough invariant geometry survives.
