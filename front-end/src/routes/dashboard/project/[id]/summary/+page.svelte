<script lang="ts">
    import Card from "$lib/components/Card.svelte";
    import { createClient } from "@supabase/supabase-js";
    import { onMount } from "svelte";
    import { page } from "$app/state";

    interface Props {
        testResults: any[];
    }

    onMount(async () => {
        let { data: d1, error } = await supabase
            .from("actions")
            .select("*")
            .eq("project_id", page.params.id)
            .eq("agent", "hacker")
            .order("created_at", { ascending: false })
            .limit(1);

        hackerSummary = d1[0] || {};

        let { data: d2, error: e2 }
            = await supabase
            .from("actions")
            .select("*")
            .eq("project_id", page.params.id)
            .eq("agent", "boomer")
            .order("created_at", { ascending: false })
            .limit(1);

        boomerSummary = d2[0] || {};

        let { data: d3, error: e3 }
            = await supabase
            .from("actions")
            .select("*")
            .eq("project_id", page.params.id)
            .eq("agent", "accessibility cop")
            .order("created_at", { ascending: false })
            .limit(1);

        accessibilityCopSummary = d3[0] || {};

        let { data: d4, error: e4 }
            = await supabase
            .from("actions")
            .select("*")
            .eq("project_id", page.params.id)
            .eq("agent", "geek")
            .order("created_at", { ascending: false })
            .limit(1);

        powerUserSummary = d4[0] || {};
    });

    const supabase = createClient(
        "https://ttnxgveqyvtsjlwvpecl.supabase.co",
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InR0bnhndmVxeXZ0c2psd3ZwZWNsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTA1NDUwNzgsImV4cCI6MjA2NjEyMTA3OH0.EkiiotOoeCCpt5kY2_oSVVTCm_7M9IXoNWUt5O8WiwQ"
    );

    let hackerSummary = $state({});
    let boomerSummary = $state({});
    let accessibilityCopSummary = $state({});
    let powerUserSummary = $state({});

    let { testResults }: Props = $props();

    const getVerdictColor = (verdict: string) => {
        switch (verdict) {
            case "pass":
                return "bg-green-100 text-green-800 border-green-200";
            case "partial":
                return "bg-orange-100 text-orange-800 border-orange-200";
            case "fail":
                return "bg-red-100 text-red-800 border-red-200";
            default:
                return "bg-gray-100 text-gray-800 border-gray-200";
        }
    };

    // Helper: map agent name to human-friendly name
    const agentToHumanFriendlyName = (agent) => {
        if (agent === "geek") return "Power User";
        else if (agent === "boomer") return "Grandpa Joe";
        else if (agent === "accessibility cop") return "Accessibility Officer";
        else if (agent === "hacker") return "The Hacker";
        else return agent;
    };

    // Group actions by agent and get the last action with finalVerdict for each
    let finalResults = $state([]);
    $derived: if (testResults && testResults.length) {
        const byAgent = {};
        for (const action of testResults) {
            if (action.finalVerdict) {
                if (!byAgent[action.agent] || (action.created_at > byAgent[action.agent].created_at)) {
                    byAgent[action.agent] = action;
                }
            }
        }
        finalResults = Object.values(byAgent);
    }
</script>

<div class="space-y-6">
  <Card>
    <h3 class="text-xl font-semibold text-gray-900 mb-6">Overall Summary</h3>
      <!-- Individual Summaries -->
      <div class="mt-6">
        <h4 class="font-semibold text-gray-900 mb-4">Final Persona Results</h4>
        <!--                Show each summary for each persona-->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            {#each [hackerSummary, boomerSummary, accessibilityCopSummary, powerUserSummary] as agentSummary}
            <div class="bg-white/20 rounded-xl p-4 border border-white/40">
                <h5 class="font-semibold text-gray-900 mb-2">{agentToHumanFriendlyName(agentSummary.agent)}</h5>
                <p class="text-gray-700 mb-2">{agentSummary.finalSummary || "No summary available."}</p>
                {#if agentSummary.finalIssues && (Array.isArray(agentSummary.finalIssues) ? agentSummary.finalIssues.length > 0 : (typeof agentSummary.finalIssues === 'string' && agentSummary.finalIssues.length > 2))}
                <div class="mb-2">
                    <div class="font-semibold text-[#d83458] mb-1">Final Issues</div>
                    <ul class="list-disc list-inside text-[#DC5270] text-sm">
                    {#each (
                        (() => {
                            let issues = agentSummary.finalIssues;
                            if (typeof issues === 'string') {
                                try {
                                    const parsed = JSON.parse(issues);
                                    if (Array.isArray(parsed)) return parsed.flatMap(i => typeof i === 'string' ? i.split('\n') : [i]).filter(Boolean);
                                } catch { /* not JSON */ }
                                return issues.split('\n').filter(Boolean);
                            } else if (Array.isArray(issues)) {
                                return issues.flatMap(i => typeof i === 'string' ? i.split('\n') : [i]).filter(Boolean);
                            } else {
                                return [issues];
                            }
                        })()
                    ) as issue}
                        <li>{issue}</li>
                    {/each}
                    </ul>
                </div>
                {/if}
                {#if agentSummary.finalRecommendations && (Array.isArray(agentSummary.finalRecommendations) ? agentSummary.finalRecommendations.length > 0 : (typeof agentSummary.finalRecommendations === 'string' && agentSummary.finalRecommendations.length > 2))}
                <div class="mb-2">
                    <div class="font-semibold text-[#34a0c1] mb-1">Recommendations</div>
                    <ul class="list-disc list-inside text-[#6DBDD5] text-sm">
                    {#each (
                        (() => {
                            let recs = agentSummary.finalRecommendations;
                            if (typeof recs === 'string') {
                                try {
                                    const parsed = JSON.parse(recs);
                                    if (Array.isArray(parsed)) return parsed.flatMap(i => typeof i === 'string' ? i.split('\n') : [i]).filter(Boolean);
                                } catch { /* not JSON */ }
                                return recs.split('\n').filter(Boolean);
                            } else if (Array.isArray(recs)) {
                                return recs.flatMap(i => typeof i === 'string' ? i.split('\n') : [i]).filter(Boolean);
                            } else {
                                return [recs];
                            }
                        })()
                    ) as rec}
                        <li>{rec}</li>
                    {/each}
                    </ul>
                </div>
                {/if}
                <p class="text-sm text-gray-500 mt-1">Last updated: {agentSummary.created_at ? new Date(agentSummary.created_at).toLocaleString() : 'N/A'}</p>
            </div>
            {/each}
        </div>
      </div>
  </Card>
</div>
