# TSC Submission : Merkle proof standardised format

|Attribute|Description
|---|---|
| Version| 0.1 |
| Authors | Steve Shadders |
| Reviewers | Joe Bloggs |
| Publication Date | dd-mm-yyyy |
| Risk Categories | xxxxxx |
| Copyright | xxxxx |
| IP Generation | xxxxx |
| Known Implementations | xxxxxxxx |
| Applies To | Miners, Wallets,                 Service Providers |
| BRFC ID | n/a |
| Acknowledgements | xxxxxxxx |
| Status | Submission |
| Visibility | IN-CONFIDENCE |



# Title 

## Merkle proof standardised format

## Background

The merkle proof is a fundamental to the SPV verification model that underpins bitcoin scaling. In order for SPV wallets to make use of them they need to be provided to wallets by miners or other actors to download full blocks. As there is likely to be a wide variety of implementations transmitting and receiving merkle proofs for various reasons it simplifies implementations if a standard format is adopted.


4:              root
            /           \
3:        c               p
       /     \         /     \
2:    p       c       o       o
     / \     / \     / \     / \
1:  o   o   c   p   o   o   o   o
   /\  /\  /\  /\  /\  /\  /\  /\
0: o o o o c p o o o o o o o o o o
           ^
           |
           tx
```

## Execution algorithm

The outcome of the algorithm is a proof that a given transaction ID is included in a given block header. We do not address the validity of the block header itself in this standard, only whether the the proof of inclusion is valid.

Although some of the following combinations may not have real use cases, for future proofing the following version values are reserved.

### Index and left/right determination

A complete merkle proof requires not only the transaction you are proving but also the transaction ID of it's pair.  If the transaction index in layer 0 of the tree is `i` then where `i mod 2 == 0` the pair will be at index `i + 1`.  Where `i mod 2 == 1` the pair will be at index `i - 1`. This ordering is required to be maintain when hashing the left/right nodes to calculate the parent. It can optionally also be provided as a flag for each provided node in the tree.

We have chosen to use the positional index rather than left/right flags for the following reasons:
1. It simplifies the data structure to include only a single integer
2. The index in block *may* be useful in some cases
3. The index in block can be proven as part of the merkle proof
4. Calculation of the leftness of rightness of a node along with the indexes of parent nodes during proof execution is trivially simple.
5. Whilst it is possible to calculate the positional index using left right flags, the calculation is more complex and even in the best cases (assuming 1TB blocks) requires just as much data to be included as is required for the positional index.

### Nodes

The `nodes` field is an array data structure that contains all of the nodes that must be provided as pairs to the calculated nodes.  It does NOT include the transaction ID being proven (this is assumed to be calculated from the transaction) OR the merkle root (this is calculated as part of the proof).

Both the binary and the JSON formats specify a mechanism for designating elements of the path that are duplicates. That is, where the provided element `p[n]` should be assumed to be a copy of of the calculated element `c[n]` for that layer of the tree.

## Merkle proof validation

In the following example we will validate a simple merkle branch type proof.

``` javascript

//note that the function sha256d(bytes) is assumed to return reversed endian bytes

let layers = nodes.length + 1; //number of layers in tree
let merkle_root = resolveFromTarget(flags, target); //function not provided
let txid = resolveFromTxOrId(flags, txOrId); //function not provided
let n = 0; //current layer
let i = index; //index of node in current layer
let c = txid;
let is_last_in_tree = true;

foreach(node in nodes) {

    
    let c_is_left = i % 2 == 0;
    let p = nodes[n];
    
    //Check for duplicate hash
    if (p == "*") {
        p = c;
    }

    //This check fails at least once if positional index is valid
    if (p == c && !c_is_left) {
        //this should happen so we can fail here
        throw 'left node cannot be a copy';
    }

    //This check fails at least once if it's not the last element
    if (c_is_left && c != p) {
        is_last_in_tree = false;
    }  

    //Calculate the parent node
    let parentC = c_is_left? sha256d(concat(c, p)) : sha256d(concat(p, c));
    c = parentC;

    //Update index for next layer
    n++;
    //We need integer division here with remainder dropped.
    //Javascript does floating point math by default so we
    //need to use Math.floor to drop the fraction.
    //In most languages we would use: i = i / 2;
    i = Math.floor(i / 2);
}

//c is now the calculated merkle root
let is_proof_valid = c == merkle_root

```

## History

|Artefact|Description
|---|---|
| Errata| n/a |
| Change Log | n/a |
| Decision Log |n/a |

## Relationships

|Relationship|Description
|---|---|
| IP licences and dependencies| n/a |
| Previous versions | n/a |
| Extends (existing standards extended by this standard) |n/a |
| Modifies (existing standards changed by this standard)| n/a |
| Deprecates (existing standards obsoleted by this standard) | n/a |
| Depends On (existing standards that are foundational or prerequisite to this standard)|n/a |
| Prior Art| n/a |
| Existing Solution | n/a |
| References |n/a |

## Supplementary Material

No supplementary material