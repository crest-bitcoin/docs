# Crest Documentation

Technical documentation for Crest - A self-custodial Bitcoin trading platform powered by RFQ on Citrea.

## Overview

This repository contains comprehensive documentation for the Crest protocol, including:

- **Introduction**: What Crest is and how it works
- **Architecture**: Deep dive into the RFQ system and smart contracts
- **Smart Contracts**: Detailed contract references and code examples
- **Trading Mechanisms**: RFQ-T and RFQ-M settlement modes
- **Integration Guides**: For market makers
- **Security**: Best practices and security considerations

## Built with Mintlify

This documentation is built using [Mintlify](https://mintlify.com), providing a beautiful and interactive documentation experience.

## Local Development

To run the documentation locally:

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

The documentation will be available at `http://localhost:3000`.

## Building

To build the documentation for production:

```bash
npm run build
```

## Key Features Documented

### Core Protocol
- **Ping-based RFQ System**: 500ms quote response window with parallel market maker pinging
- **Native Bitcoin Support**: Direct cBTC trading on Citrea without custodial bridges
- **Dual Settlement Modes**: RFQ-T (user-initiated) and RFQ-M (relayer-initiated)
- **WCBTC Integration**: Seamless wrapping/unwrapping of native Bitcoin

### Smart Contracts
- **Settlement Contract**: Core trading settlement with multi-signature validation
- **WCBTC Contract**: Wrapped cBTC for ERC20 compatibility
- **Gas Optimization**: Efficient contract design for low-cost trading

### Integration Support
- **Market Maker APIs**: WebSocket protocols and quote generation
- **Frontend SDKs**: React components and trading interfaces
- **Security Guidelines**: Best practices for safe integration

## Contributing

To contribute to the documentation:

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Links

- **Protocol**: [https://crest.finance](https://crest.finance)
- **App**: [https://app.crest.finance](https://app.crest.finance)
- **GitHub**: [https://github.com/crest-protocol](https://github.com/crest-protocol)
- **Discord**: [https://discord.gg/crest](https://discord.gg/crest)

## License

This documentation is licensed under MIT License.