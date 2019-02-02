# Block and transaction validation

## Overview

Validation of transactions is the most critical task of stakeholders and auditors, and therefore we very rigorously specify the process. Validation is done in two contexts in a typical node implementation:

* **Preblock building**: between blocks, newly seen transactions are validated in an online fashion and placed into a "mempool" or "preblock" as a candidate for the next block.
* **Block validation**: transactions in a block are batch-validated during the consensus process, and during relaying by auditors.

## Incremental validation

Incremental validation is done when 

 

