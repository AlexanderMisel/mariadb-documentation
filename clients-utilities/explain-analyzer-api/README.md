# EXPLAIN Analyzer API

The online [EXPLAIN Analyzer](/clients-utilities/explain-analyzer/) tool has an open API to allow client applications to send it EXPLAINs.

## Sending EXPLAINs to the EXPLAIN Analyzer

To send an EXPLAIN to the EXPLAIN Analyzer, simply POST or GET to the following address:

```sql
mariadb.org/explain_analyzer/api/1/?raw_explain=EXPLAIN&client=CLIENT
```

Replace "EXPLAIN" with the output of the EXPLAIN command and "CLIENT" with the name of your client.

## Client Banner

If you like, you can have a banner promoting your client appear at the bottom of the page. Once you've added support for the EXPLAIN Analyzer to your client application, just send a logo, the name of your client, and what you want the name and logo to link to to bryan AT montyprogram DOT com