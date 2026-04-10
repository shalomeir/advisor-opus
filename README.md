# ⚠️ Deprecated — Advisor Opus

> **Claude Code에 `/advisor` 기능이 공식 출시(혹은 출시 예정)되어, 이 플러그인은 deprecated 되었습니다.**
>
> 이 플러그인이 제공하던 Advisor 패턴이 Claude Code에 네이티브로 내장됩니다.
> **공식 기능을 사용하세요:** Claude Code 세션에서 `/advisor`를 입력하면 됩니다.
>
> 다만, 이 플러그인은 **skill 호출 시점을 직접 커스터마이즈**할 수 있어 여전히 유용합니다.
> Repo는 계속 유지되며, 설치하여 사용할 수 있습니다.

---

# Advisor Opus — Claude Code Plugin

> **This is an unofficial, community-driven plugin.** Not affiliated with or endorsed by Anthropic.

Sonnet이나 Haiku 세션에서 모델 전환 없이, 결정적인 순간에만 Opus의 전략적 사고를 활용하는 Claude Code 플러그인입니다.

**Use Opus's brain at critical moments — without switching your session.**

## Why

Anthropic's [Advisor Strategy](https://claude.com/blog/the-advisor-strategy): pair a fast executor (Sonnet/Haiku) with a higher-intelligence advisor (Opus) at key decision points.

**Most work is mechanical. But a wrong direction discovered 30 minutes in costs far more than a 2-minute Opus call that catches it upfront.**

## Install

```bash
/plugin marketplace add shalomeir/advisor-opus
/plugin install advisor-opus@advisor-opus
/reload-plugins
```

## When Does It Invoke?

설치하면 Sonnet/Haiku 세션에서 다음 시점에 **자동으로 Opus advisor 호출을 제안**합니다:

| 시점 | Skill | 트리거 조건 |
|---|---|---|
| **PLANNING** | `/advisor-opus:plan` | 실행 직전, 복잡한 작업(3+ 단계, 멀티 파일 변경)일 때 |
| **COMPLETION** | `/advisor-opus:review` | 코드 작성 완료 후, 커밋 직전 (파일 작성됨, 테스트 실행됨) |
| **PIVOT / REACTIVE** | `/advisor-opus:advise` | 방향 전환이 필요하거나 막혔을 때 (드묾) |

- 목표: **복잡한 작업당 2회, 최대 4회** (planning 1회 + completion 1회)
- 단순한 작업은 항상 스킵
- Opus 세션에서는 auto-invocation 비활성화 (같은 모델이 자기 자신에게 조언하는 건 의미 없음)
- 수동 호출(`/advisor-opus:plan`, `/advise`, `/review`)은 언제든 가능

## Commands

### `/advisor-opus:plan`
Your session gathers codebase context and forwards it to Opus for a strategic plan. Output: End State → Critical Path → Risks → First Move.

```bash
/advisor-opus:plan add OAuth2 authentication with Google and GitHub
/advisor-opus:plan migrate REST API from Express to Hono
```

### `/advisor-opus:advise`
Get strategic advice when you're stuck or want a second opinion on direction.

```bash
/advisor-opus:advise should I use a queue or a cron job for this batch processing?
/advisor-opus:advise tests pass locally but fail in CI — what to investigate first?
```

### `/advisor-opus:review`
Final review before commit. Checks correctness, architecture, security, and silent failures.

```bash
/advisor-opus:review
/advisor-opus:review the authentication middleware in src/middleware/auth.ts
```

## Which Model Benefits Most

| Your session | Recommendation | Why |
|---|---|---|
| **Haiku** | Strongly recommended | Largest intelligence gap — Opus guidance dramatically improves output |
| **Sonnet** | Recommended (sweet spot) | Meaningful upgrade at critical moments, bulk work stays at Sonnet cost |
| **Opus** | Not recommended | No intelligence gap — same model advising itself. Just ask your session to "step back and reconsider" instead. Auto-invocation is disabled. |

## Cost

Each advisor call spawns a full Opus sub-agent — this **adds** to your session cost.

The value is **preventing wasted work**: catching wrong approaches and missed edge cases before they cost far more tokens to fix. Whether it nets positive depends on task complexity — multi-step work with real risk of wrong direction benefits most; trivial tasks don't need it.

Auto-invocation targets **2 calls per complex task, max 4**: one before execution (PLANNING), one before commit (COMPLETION). Trivial tasks are always skipped. Manual `/plan`, `/advise`, `/review` bypass auto-invocation gates (still incurs Opus cost).

**This is not "cheaper Opus."** It's a small, targeted investment in Opus-level judgment at critical decision points.

## References

- [The Advisor Strategy](https://claude.com/blog/the-advisor-strategy) — Anthropic blog
- [Advisor Tool Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) — API reference

## License

MIT
