# AWS EKS Observability Cost Optimization Pattern

## 📖 Overview
This repository documents a critical architectural pattern for reducing Amazon EKS monitoring costs on AWS CloudWatch. 

This pattern was discovered during a production audit where enabling **Enhanced Container Insights** paradoxically reduced costs by **21%** (despite a 30x increase in metric volume) by leveraging the `ObservationUsage` billing tier instead of the legacy `CustomMetrics` tier.

## 🏗 The Architecture

### The Problem (Legacy Mode)
* **Configuration:** Standard CloudWatch Agent / FluentBit.
* **Data Flow:** Logs -> CloudWatch Logs -> Metric Filters -> Custom Metrics.
* **Billing Impact:** High costs due to "Data Processing" (EMF) and "Custom Metric" per-metric charges.

### The Solution (Enhanced Mode)
* **Configuration:** `enhanced_container_insights = enabled`
* **Data Flow:** Structured Performance Logs -> CloudWatch Container Insights.
* **Billing Impact:** * **Data Processing:** $0 (Native ingestion).
    * **Metric Storage:** Billed as "Observation Usage" (Bulk tier).

![Cost Optimization Flow](./cost-optimization-flow.png)

## 💻 Implementation Reference

To implement this pattern in Terraform using the `aws-ia/eks-blueprints-addons/aws` or standard `helm_release`, ensure the following configuration is applied to your CloudWatch Observability add-on:

```hcl
# Terraform Reference Configuration
resource "aws_eks_addon" "cloudwatch_observability" {
  cluster_name = var.cluster_name
  addon_name   = "amazon-cloudwatch-observability"
  
  configuration_values = jsonencode({
    container_logs = {
      enabled = true
    }
    # THE CRITICAL CHANGE
    # Switching this to true enables the ObservationUsage billing model
    enhanced_container_insights = true 
  })
}
```

🔗 References

Full Case Study: https://www.xgrid.co/resources/how-we-got-30x-more-eks-metrics-while-slashing-client-costs/

AWS Documentation: https://aws.amazon.com/blogs/aws/container-insights-with-enhanced-observability-now-available-in-amazon-ecs/

Maintained by Hamza Shaukat
