# NOSTR Network Analysis

A research project for analyzing the **NOSTR** (Notes and Other Stuff Transmitted by Relays) decentralized network. The project analyzes over 200 million events and nearly 1 billion event-relay associations to understand the structure, user behavior, and network topology.

## Dataset

The PostgreSQL database contains:

| Table | Rows | Size | Description |
|-------|------|------|-------------|
| **events** | 208.5M | 219.51 GB | All collected NOSTR events |
| **events_relays** | 976.4M | 280.86 GB | Event-relay associations with seen_at timestamp |
| **relays** | 8,865 | ~0.01 GB | Known relay URLs (clearnet + tor) |
| **relay_metadata** | 74,274 | 0.05 GB | NIP-11 metadata snapshots over time |

**Total size: ~500 GB** (data) + **~233 GB** (indexes) = **~1.26 TB**

### Storage Breakdown
- **events**: 179 GB data + 40.5 GB indexes
- **events_relays**: 89 GB data + 192 GB indexes (primary key index alone is 144 GB)
- **content** field averages 357 bytes, **tags** field averages 290 bytes per event

## Project Structure

```
analysis/
├── lib/                    # Core NOSTR libraries
│   ├── event.py            # Event class (validation and signing)
│   ├── relay.py            # WebSocket relay management
│   ├── relay_metadata.py   # NIP-11 metadata
│   └── utils.py            # Cryptographic utilities
├── analysis/
│   ├── src/                # Jupyter notebooks
│   ├── pdf/                # PDF reports
│   └── html/               # HTML reports
├── utils/                  # Utility scripts
└── image/                  # Key visualizations
```

## Main Analyses

The Jupyter notebooks in `analysis/src/` cover:

- **Database Overview**: Schema, space usage, indexes
- **Events Overview**: Temporal distribution, event types, pubkey behavior
- **Relays Overview**: Connectivity, RTT, NIP-11 metadata, clearnet vs tor comparison
- **Events-Relays Correlation**: Event replication, relay coverage and resilience
- **Pubkeys Analysis**: K-means user clustering (10 behavioral profiles)

## Key Findings

### Event Distribution

| Kind | Name | Count | % of Events | Unique Users |
|------|------|-------|-------------|--------------|
| 1 | Short Text Note | 67.3M | 38.96% | 3.4M |
| 7 | Reaction | 32.6M | 18.89% | 218K |
| 4 | Encrypted DMs | 9.7M | 5.63% | 449K |
| 0 | User Metadata | 8.2M | 4.73% | 7.4M |
| 3 | Follows | 7.8M | 4.54% | 7.3M |
| 6 | Repost | 7.6M | 4.40% | 100K |
| 5 | Event Deletion | 5.4M | 3.12% | 119K |
| 9735 | Zap | 4.9M | 2.81% | - |
| 1059 | Gift Wrap | 3.8M | 2.17% | 3.7M |
| 10002 | Relay List Metadata | 3.2M | 1.88% | 1.9M |

**Key insights**:
- **77.6%** of all pubkeys have published at least one of kinds 0, 1, 3, 6, or 7
- Only **17.98%** of pubkeys actively post/repost/react (kinds 1, 6, 7)
- **Top 1% of users** generate 44% of text notes and 23% of reactions
- **Single-event pubkeys** mostly publish kind 3 (follows, 37%), kind 0 (metadata, 29%), or kind 1059 (gift wrap, 25%)

### User Behavior Patterns

- **19.7M unique pubkeys** observed across 172.7M events (filtered dataset)
- **~80% of pubkeys** have a lifespan < 1 second (single activity)
- **Stable users** (lifespan > 30 days) generate 70-90% of monthly events
- **Ephemeral accounts** (< 1 day) spiked to 56-57% during spam waves (June/November 2024)

### Relay Infrastructure

**Network composition** (of 1,247 relays with metadata):
- **Clearnet**: 997 relays (80%)
- **Tor**: 250 relays (20%)

**Relay health metrics**:
| Metric | Clearnet | Tor | Overall |
|--------|----------|-----|---------|
| Connection success | 98.5% | 99.6% | 98.7% |
| NIP-11 available | 91.4% | 99.6% | 93.0% |
| Readable | 73.8% | 98.8% | 78.8% |
| Writable | 44.5% | 20.5% | 39.7% |

**Performance** (RTT median, filtered 5-95 percentile):
- **Clearnet**: ~350ms open, ~250ms read, ~150ms write
- **Tor**: ~870ms open (+157%), ~550ms read (+127%), ~520ms write (+246%)

**Relay restrictions**:
- **52.8%** require payment
- **40.3%** have restricted writes
- Tor relays are predominantly read-only with restricted writes (73% vs 7% clearnet)

### Event Replication & Resilience

- **Top 20 relays** cover 63% of all events and 48% of all pubkeys
- **Top 100 relays** reach ~95% coverage for both events and pubkeys
- Removing top 200 relays still leaves ~10% of events accessible elsewhere
- **42% of events** appear on only 1 relay; **~60% of pubkeys** use only 1 relay
- Long-lived users (>30 days) achieve faster coverage across relays than ephemeral accounts

### Top Relays by Volume

**By event count**:
1. `relay.nostr.band` - 45M events
2. `a.nos.lol` - 19.5M events
3. `nos.lol` - 12.6M events

**By unique pubkeys**:
1. `directory.yabu.me` - 7.5M pubkeys
2. `relay.nos.social` - 6.6M pubkeys
3. `nostr.oxtr.dev` - 4.4M pubkeys

**Tor hidden services** (mirror same data as clearnet):
- `oxtrdevav64z64yb7x...onion` mirrors `nostr.oxtr.dev` (8.8M events, 4.4M pubkeys)
- `sovbitm2enxfr5ot6q...onion` mirrors `freelay.sovbit.host` (7.4M events, 1.1M pubkeys)

## Key Visualizations

### 1. Monthly Proportion of Events by Kind

![Events by Kind Heatmap](image/events-proportion-by-kind-heatmap.png)

This heatmap shows the monthly distribution of event proportions for each NOSTR event type (kind) from December 2022 to July 2025. The most significant types are:

- **Kind 1** (Short Text Notes): Consistently dominant with proportions between 0.30-0.70, representing the main text posts
- **Kind 7** (Reactions): Second most frequent (0.13-0.44), indicating high engagement
- **Kind 4** (Encrypted DMs): Occasional spikes, such as 0.24 in November 2023
- **Kind 3** (Contact Lists): Notable peak of 0.52 in June 2024
- **Kind 10002** (Relay List Metadata): Emerged in 2024, growing to 9% by mid-2025

Seasonal patterns and the emergence of new types (9041, 10002) in 2024-2025 are observed.

### 2. Daily and Monthly Active Users

![Daily and Monthly Active Users](image/daily-monthly-active-users.png)

Logarithmic scale chart showing the evolution of active users:

- **Blue line (DAU)**: Daily active users, oscillating between 10³ and 10⁵
- **Green line (MAU)**: Monthly active users (rolling sum), consistently above 10⁵

**Key observations**:
- Initial exponential growth (Dec 2022 - Mar 2023) during NOSTR's initial hype
- Stabilization in 2023 with DAU ~10-50k
- Significant spike in June 2024 (DAU > 10⁶)
- Sustained growth trend in 2025 with DAU consistently reaching 10⁵

### 3. Event Distribution by Pubkey Lifespan

![Events by Pubkey Lifespan Heatmap](image/events-by-pubkey-lifespan-heatmap.png)

Heatmap classifying pubkeys into three "lifespan" categories (time between first and last event):

- **< 1 day**: Ephemeral accounts, often bots or occasional users
- **1-30 days**: Temporary users or those who tried and abandoned
- **> 30 days**: Stable and persistent users

**Key insights**:
- Stable users (>30 days) generate 70-90% of events in most months
- Spikes in ephemeral accounts (<1 day) correlate with spam events (June 2024: 0.56, November 2024: 0.57)
- The 1-30 days category peaked at 48% in February 2023, indicating many early adopters who didn't stay

### 4. Cumulative Growth and Distribution by Lifespan

![Cumulative Events and Pubkeys by Lifespan](image/cumulative-events-pubkeys-by-lifespan.png)

Combined chart showing:

- **Blue CDF**: Cumulative distribution function of total events
- **Green CDF**: Cumulative distribution function of total pubkeys
- **Stacked histogram**: Daily events colored by pubkey lifespan

**Analysis**:
- Pubkey growth (green) precedes event growth (blue), indicating new users initially explore before becoming active
- Spikes in the red histogram (< 1 day) indicate spam or bot waves
- Predominant green in 2025 indicates a more mature network with stable users
- Growth visibly accelerates from late 2024, suggesting increasing adoption

### 5. Relay Coverage by Top N Relays

![Relay Coverage by Top N Relays](image/relay-coverage-by-top-n.png)

This chart shows cumulative coverage of events and pubkeys as more relays are added (ranked by volume):

- **Blue line**: Event coverage percentage
- **Orange line**: Pubkey coverage percentage

**Key observations**:
- **Top 20 relays** cover ~63% of events and ~48% of pubkeys
- **Top 100 relays** reach ~95% coverage for both metrics
- Coverage flattens after ~200 relays, with the long tail contributing marginally
- Pubkey coverage grows faster initially, indicating users are more concentrated on popular relays
- Full coverage requires ~600+ relays, demonstrating the decentralized nature of NOSTR

### 6. Pubkey Distribution by Event Kind

![Venn Diagram of Pubkeys by Event Kind](image/venn-diagram-pubkeys-by-kind.png)

Proportional Venn diagram showing overlap between pubkeys publishing different event types:

- **kind_0** (red): User metadata - 7.4M pubkeys
- **kind_3** (green): Follows/contact lists - 7.3M pubkeys
- **kind_1_6_7** (blue): Active users (notes, reposts, reactions) - 3.5M pubkeys

**Key insights**:
- **5.0M pubkeys** have only metadata (kind 0) without any follows or activity
- **5.5M pubkeys** have only follows (kind 3) without metadata or activity
- **2.3M pubkeys** are active (kind 1/6/7) without metadata or follows
- Only **548K pubkeys** (~3.6% of active users) have all three: metadata, follows, AND activity
- **1.17M pubkeys** have both metadata and follows but no posts/reactions
- This reveals that most NOSTR accounts are incomplete profiles or passive observers

### 7. CDF of Events per Pubkey

![CDF of Events per Pubkey](image/cdf-events-per-pubkey.png)

Cumulative Distribution Function showing the concentration of activity:

- **X-axis**: Number of events per pubkey (log scale)
- **Y-axis**: Cumulative proportion of pubkeys
- **Inset**: Zoomed view of the top 10% most active users

**Key observations**:
- **~75% of pubkeys** have published only 1 event
- **~90% of pubkeys** have fewer than 10 events
- **~99% of pubkeys** have fewer than 1,000 events
- The top 1% of users (with 1,000+ events) generate the vast majority of content
- Some power users have over 10^6 events, indicating heavy automation or bot activity
- This extreme power-law distribution is typical of social networks

### 8. RTT Comparison: Clearnet vs Tor

![RTT Comparison Clearnet vs Tor](image/rtt-comparison-clearnet-tor.png)

Bar chart comparing Round Trip Time (RTT) between clearnet and Tor relays for different operations:

- **Blue**: Connection open time
- **Orange**: Read operation time
- **Green**: Write operation time

**Performance comparison** (filtered 5-95 percentile):
| Operation | Clearnet | Tor | Overhead |
|-----------|----------|-----|----------|
| Open | ~340ms | ~870ms | +157% |
| Read | ~230ms | ~520ms | +127% |
| Write | ~120ms | ~420ms | +246% |

**Key insights**:
- Tor adds significant latency due to onion routing (3+ hops)
- Write operations suffer the most (+246%), making Tor relays less suitable for real-time posting
- Read operations have the smallest overhead (+127%), making Tor viable for content consumption
- Despite higher latency, Tor relays offer privacy benefits for users in restrictive environments

### 9. User Clustering (K-means with k=10)

The pubkey analysis identifies 10 distinct user behavioral clusters based on:
- Event count, lifespan, posting intervals
- Follower/following counts
- Number of read/write relays used

**Cluster characteristics**:
- **Clusters 0, 1, 8**: High-activity users with long lifespans (~1 year), many events (10³-10⁶), and high follower counts
- **Clusters 2, 3**: Short-lived accounts with minimal activity (bots or abandoned accounts)
- **Cluster 4**: Power users with extremely high event counts (10⁵-10⁶), used since 2023
- **Cluster 5, 6**: Recent joiners (2024-2025) with moderate activity and short lifespans
- **Cluster 7**: Early adopters (2022-2023) with moderate but consistent activity
- **Cluster 9**: New users (late 2024-2025) with medium activity levels

This clustering helps identify bots, power users, casual users, and spam accounts.

## Technologies

- **Database**: PostgreSQL
- **Analysis**: Python, Pandas, Polars, NumPy, SciPy
- **Visualization**: Matplotlib, Seaborn
- **ML**: scikit-learn (K-means clustering)
- **Cryptography**: secp256k1, bech32

## Setup

```bash
# Clone the repository
git clone https://github.com/username/analysis.git
cd analysis

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp env.example .env
# Edit .env with database credentials

# Start Jupyter
jupyter lab
```

## License

See [LICENSE](LICENSE) for details.
