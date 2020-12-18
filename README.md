
This module is based on the golang `crypto/tls` package (more specifically, at the point of the commit https://github.com/golang/go/commit/8fcf318123e15abf6ce35e33831bdb64a4e071ff).
The tls package was modified to expose parameters that were not modifiable normally
and to support changing the order of the extensions in the `ClientHello` message.

This was done in order to defeat tls fingerprint checks that some web services perform,
such as the Google Android Authentication API.


Do not use this module for security sensitive work because I may have introduced vulnerabilities in to
the tls package.

The following parameters were added into the `tls.Config` struct:
```
CompressionMethods []uint8
Scts                         bool
OscpStapling                 bool
SupportedCurves              []CurveID
SupportedPoints              []uint8
SecureRenegotiationSupported bool
SupportedSignatureAlgorithms []SignatureScheme
SupportedVersions            []uint16
ClientHelloVersion           uint16
TicketSupported              bool
PskModes                     []uint8
Extensions                   []uint16 // Extensions are appended on this order to the message
RawClientHello	[]byte // If set, uses this instead of constructing ClientHello based on the configured parameters
```

Usage example:

`````golang
package main

import (
	xtls "github.com/Jarijaas/go-tls-exposed/tls"
	"log"
)

func main() {
	conf := &xtls.Config{
		CipherSuites: []uint16{
			0x1302, 0x1303, 0x1301, 0xc02c,
			0xc030, 0xc02b, 0xc02f, 0xcca9,
			0xcca8, 0x00a3, 0x009f, 0x00a2,
			0x009e, 0xccaa, 0xc0af, 0xc0ad,
			0xc024, 0xc028, 0xc00a, 0xc014,
			0xc0a3, 0xc09f, 0x006b, 0x006a,
			0x0039, 0x0038, 0xc0ae, 0xc0ac,
			0xc023, 0xc027, 0xc009, 0xc013,
			0xc0a2, 0xc09e, 0x0067, 0x0040,
			0x0033, 0x0032, 0x009d, 0x009c,
			0xc0a1, 0xc09d, 0xc0a0, 0xc09c,
			0x003d, 0x003c, 0x0035, 0x002f,
			0x00ff,
		},
		TicketSupported:   true,
		PskModes:          []uint8{xtls.PskModeDHE},
		SupportedVersions: []uint16{xtls.VersionTLS13, xtls.VersionTLS12},
		SupportedSignatureAlgorithms: []xtls.SignatureScheme{
			0x0403, 0x0503, 0x0603, 0x0807, 0x0808, 0x0809, 0x080a, 0x080b, 0x0804, 0x0805,
			0x0806, 0x0401, 0x0501, 0x0601, 0x0303, 0x0301, 0x0302, 0x0402, 0x0502, 0x0602,
		},
		OscpStapling:                 true,
		Scts:                         true,
		CompressionMethods:           []uint8{xtls.CompressionNone},
		SecureRenegotiationSupported: false,
		ClientHelloVersion:           xtls.VersionTLS12,
		SupportedPoints:              []uint8{xtls.PointFormatUncompressed, 1, 2},
		SupportedCurves:              []xtls.CurveID{0x001d, 0x0017, 0x001e, 0x0019, 0x0018},
		Extensions: []uint16{
			xtls.ExtensionServerName, xtls.ExtensionSupportedPoints, xtls.ExtensionSupportedCurves,
			xtls.ExtensionSessionTicket, xtls.ExtensionEncryptThenMac, xtls.ExtensionExtendedMasterSecret,
			xtls.ExtensionSignatureAlgorithms, xtls.ExtensionSupportedVersions, xtls.ExtensionSignatureAlgorithmsCert,
			xtls.ExtensionPSKModes, xtls.ExtensionKeyShare,
		},
	}

	conn, err := xtls.Dial("tcp", "android.clients.google.com:443", conf)
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()
}
`````