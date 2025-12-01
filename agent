from typing import Any, Dict

from google.adk.agents import Agent, LlmAgent
from google.adk.apps.app import App, EventsCompactionConfig
from google.adk.models.google_llm import Gemini

from google.adk.runners import Runner
from google.adk.tools.tool_context import ToolContext
from google.genai import types

PROJECT_ID = "Default Gemini Project"  
os.environ["GOOGLE_CLOUD_PROJECT"] = PROJECT_ID

if PROJECT_ID == "your-project-id" or not PROJECT_ID:
    raise ValueError("⚠️ Please replace 'your-project-id' with your actual Google Cloud Project ID.")

print(f"✅ Project ID set to: {PROJECT_ID}")

GOOGLE_CLOUD_LOCATION="global"
GOOGLE_GENAI_USE_VERTEXAI=0

class BatchOptimizedPipeline(OptimizedCompetitorAnalysisPipeline):
    """
    For analyzing MULTIPLE competitors efficiently
    Reduce cost by batching requests
    """
    
    def analyze_multiple_competitors(self, companies: list[tuple[str, str]]) -> dict[str, str]:
        """
        Analyze multiple competitors in one go
        
        Args:
            companies: List of (website, name) tuples
        
        Returns:
            Dictionary of {company_name: html_report}
        """
        print(f"\n{'='*60}")
        print(f"🚀 BATCH Analysis for {len(companies)} competitors")
        print(f"💰 Estimated savings: ~{(len(companies) * 2)}% vs sequential")
        print(f"{'='*60}\n")
        
        # Create batch prompt for ALL companies at once
        batch_prompt = f"""
        Analyze these {len(companies)} competitors and return a JSON array with analysis for each:
        
        Companies:
        {chr(10).join([f'{i+1}. {name} - {url}' for i, (url, name) in enumerate(companies)])}
        
        Return JSON array format:
        [
            {json.dumps(COMPETITOR_ANALYSIS_SCHEMA, indent=2)},
            ... (repeat for each company)
        ]
        """
        
        response = self.unified_agent.run(batch_prompt)
        
        # Parse batch results
        try:
            if isinstance(response.content, list):
                batch_data = response.content
            else:
                json_text = response.content
                if "```json" in json_text:
                    json_text = json_text.split("```json")[1].split("```")[0]
                batch_data = json.loads(json_text.strip())
        except Exception as e:
            print(f"❌ Batch parsing failed, falling back to individual analysis")
            # Fallback to individual analysis
            results = {}
            for url, name in companies:
                html = self.analyze_competitor(url, name)
                results[name] = html
            return results
        
        # Generate HTML for each
        results = {}
        for data in batch_data:
            company_name = data.get("company_name", "Unknown")
            html = self._format_html_from_json(data)
            results[company_name] = html
            print(f"✅ Generated report for {company_name}")
        
        print(f"\n💰 Total LLM Calls: 1 (for {len(companies)} companies!)")
        return results
    
    batch_reports = batch_pipeline.analyze_multiple_competitors(competitors)
    
    for company_name, html in batch_reports.items():
        filename = f"{company_name.lower().replace(' ', '_')}_analysis.html"
        batch_pipeline.save_report(html, filename)

