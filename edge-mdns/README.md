# edge-mdns

[![CI](https://github.com/ivmarkov/edge-net/actions/workflows/ci.yml/badge.svg)](https://github.com/ivmarkov/edge-net/actions/workflows/ci.yml)
![crates.io](https://img.shields.io/crates/v/edge-net.svg)
[![Documentation](https://docs.rs/edge-net/badge.svg)](https://docs.rs/edge-net)

Async + `no_std` + no-alloc implementation of an mDNS responder.

The implementation is based on the splendid [domain](https://github.com/NLnetLabs/domain) library.

For other protocols, look at the [edge-net](https://github.com/ivmarkov/edge-net) aggregator crate documentation.

## Example

```rust
use core::net::Ipv4Addr;

use edge_mdns::io::{self, MdnsIoError, MdnsRunBuffers, DEFAULT_SOCKET};
use edge_mdns::Host;
use edge_nal::{Multicast, UdpBind, UdpSplit};

use log::*;

// Change this to the IP address of the machine where you'll run this example
const OUR_IP: Ipv4Addr = Ipv4Addr::new(127, 0, 0, 1);

const OUR_NAME: &str = "mypc";

fn main() {
    env_logger::init_from_env(
        env_logger::Env::default().filter_or(env_logger::DEFAULT_FILTER_ENV, "info"),
    );

    let stack = edge_nal_std::Stack::new();

    let mut buffers = MdnsRunBuffers::new();

    futures_lite::future::block_on(run::<edge_nal_std::Stack>(
        &stack,
        &mut buffers,
        OUR_NAME,
        OUR_IP,
    ))
    .unwrap();
}

async fn run<T>(
    stack: &T,
    buffers: &mut MdnsRunBuffers,
    our_name: &str,
    our_ip: Ipv4Addr,
) -> Result<(), MdnsIoError<T::Error>>
where
    T: UdpBind,
    for<'a> <T as UdpBind>::Socket<'a>: Multicast<Error = T::Error> + UdpSplit<Error = T::Error>,
{
    info!("About to run an mDNS responder for our PC. It will be addressable using {our_name}.local, so try to `ping {our_name}.local`.");

    let host = Host {
        id: 0,
        hostname: our_name,
        ip: our_ip.octets(),
        ipv6: None,
    };

    io::run(
        &host,
        Some(Ipv4Addr::UNSPECIFIED),
        Some(0),
        [],
        stack,
        DEFAULT_SOCKET,
        buffers,
    )
    .await
}
```
