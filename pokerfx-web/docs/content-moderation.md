# Video Upload Content Moderation

> **Status:** Considered • **Priority:** Low • **Date:** 2026-04-10

## Context

PokerFX allows users to upload poker hand videos for AI analysis and vlog processing. While the user base is poker players and content creators, we should consider the risk of CSAM (Child Sexual Abuse Material) or other harmful content being uploaded to our infrastructure.

## Risk Assessment

### Likelihood: Very Low
- Poker-specific platform with user community
- Videos are primarily poker hands, tables, cards
- No anonymity: users upload for AI processing, creating accountability
- Target audience is poker vloggers and enthusiasts

### Impact: Catastrophic
- Legal liability if CSAM is hosted
- Platform shutdown risk
- Reputation damage
- Criminal liability (18 U.S.C. § 2252A)

## Technical Solutions Considered

### Option 1: AWS Rekognition (Recommended for MVP)
**Pros:**
- Already in use (pokerfx-web uses AWS Batch, DynamoDB, S3)
- Frame-by-frame analysis with explicit content detection
- No infrastructure to manage
- Integration with existing S3 upload pipeline

**Cons:**
- Cost: ~$0.001 per frame analyzed
- False positives possible (red lighting, close-ups at poker tables)
- Latency: adds 10-30s to upload processing

**Implementation:**
```python
# Sample frames at 1fps from uploaded video
# Run Rekognition DetectModerationLabels on each frame
# Flag if confidence > 95% for explicit content
```

### Option 2: Google Cloud Video AI / Content Safety
**Pros:**
- Better accuracy on explicit content detection
- Audio analysis included
- Dedicated CSAM detection models

**Cons:**
- New dependency outside AWS stack
- Cost: ~$0.01 per minute of video
- More complex integration

### Option 3: Self-Hosted Open Source
**Tools:** YOLO-based detectors, OpenMod, etc.

**Pros:**
- Lower cost at scale
- Full control over models

**Cons:**
- GPU infrastructure required
- Maintenance burden
- Slower than managed services
- Need to train on poker-specific false positives

## Audio Analysis Component

**Why:** CSAM content often includes audio. Also helps catch harmful speech.

**Solution:**
- Whisper transcription (already considering for ASR features)
- Run text moderation on transcripts
- Flag keywords + context analysis

**Cost:** ~$0.00006 per second (Whisper API)

## Implementation Approach

### Phase 1: Frame Sampling (Lowest Cost)
1. On upload, extract frames at 1fps or keyframes only
2. Run Rekognition DetectModerationLabels
3. If any frame flagged > 95% confidence → queue for manual review
4. If clean → proceed with normal processing

### Phase 2: Audio Transcription
1. Whisper transcription of all audio
2. Text moderation on transcript
3. Flag suspicious content for review

### Phase 3: Manual Review Queue
1. Dashboard for flagged uploads
2. Human reviewer approves/rejects
3. Auto-delete rejected content after 24h

## Cost Estimate

For 1000 uploads/month, avg 5 min video each:

- **Rekognition (1fps sampling):** ~1800 frames × $0.001 = $1.80/month
- **Whisper transcription:** ~83 hours × $0.00006/sec = ~$18/month
- **Total:** ~$20/month at current scale

Scales linearly. At 10,000 uploads/month: ~$200/month

## Legal Considerations

### Safe Harbor (DMCA § 512)
- Need "actual knowledge" of infringing content
- Need to terminate repeat offenders
- Need to have DMCA agent registered

### 18 U.S.C. § 2252A
- CSAM has **no** safe harbor
- Must act immediately upon discovery
- Mandatory reporting to NCMEC
- No "fair use" defense

**Recommendation:** Register DMCA agent, add Terms of Service with prohibited content clause.

## Decision

**Status:** Defer for now

**Reasoning:**
- Current user base is poker-focused, low risk
- Cost-benefit doesn't justify implementation yet
- Can add Rekognition frame sampling as a backend job later
- Priority #1: MVP launch, #2: user growth, #3: moderation

**When to implement:**
- At 100+ daily uploads
- Or after any security incident
- Or if user base expands beyond poker community

---

## References

- [AWS Rekognition Moderation](https://docs.aws.amazon.com/rekognition/latest/dmod/labels.html)
- [Google Cloud Video AI](https://cloud.google.com/video-intelligence)
- [18 U.S.C. § 2252A](https://www.law.cornell.edu/uscode/text/18/2252A)
- [NCMEC Reporting](https://report.cybertip.org/)
- [DMCA Section 512](https://www.copyright.gov/section1201/312-notice.html)
