---
layout:		post
title:		"Restorative Justice Louisville"
image:		rjlou
tags:		[web]
priority:	4
author:     w.marsh
authorlink: https://www.flickr.com/photos/40943981@N00/2572326342/sizes/l

---
# "Junior" Design Project

By the second semester of my junior year – early 2014, to be exact – at the University of Louisville, I’d been working at Blackstone Media for almost a year, and was starting to feel like a pro at making websites. I already had a certain design style that I was proud of, and I’d written enough C# and Coldfusion at Blackstone to feel confident that I could build a solid website on my own. I got my opportunity to try my new skills out in my CIS 420 class, which allows us to build an I.T. system (99% of the time this just means a website) for a real non-profit organization in Louisville. Our project was for a group called Restorative Justice Louisville, or RJLou for short, and our class was split into three teams that each competed to build the perfect website for the client. 

While my team may not have ultimately won the challenge, the client had struggled over whether to pick [my team’s project](http://rjlou-1.apphb.com) or another’s, basically saying that she’d never seen a design as pretty as mine, but knew that the other team’s was more functional. It was a really fundamental moment in my growth as a designer, since I branched out significantly from my comfort zone and did some things that were really unique for me at the time. I also learned a lot about myself as a leader, since I ended up being the only person on my team with any real development experience, and was defaulted as the lead developer of the team. 

# Code
	
As to [the code](https://github.com/jblackett/RJLou) itself, while I was on a team of 8 people, I did end up writing most of the .aspx code completely on my own. Some of it was imitated by my teammates, and a lot of the classes were assigned to other people as well. However, I usually went back and cleaned up any of the code that I didn’t write, so almost all of it was code that I had a hand in somehow. Still, I was pretty proud to have basically taught 3/4ths of my team how to write ASP.NET, and taught the rest of the team how to architect an entire backend system. 

I was also really lucky to get an opportunity to learn a lot about how GitHub works on this project, as well as to have a teammate who helped organize a lot of this stuff. We maintained several branches for each developer, and actually handled the merges very well for college students with no guidance. It was a really cool way to be introduced to the whole platform.

[Admin/Login.aspx](https://github.com/jblackett/RJLou/blob/master/Admin/Login.aspx)
{% highlight aspx-cs %}
<%@ Page Language="C#" MasterPageFile="~/Masterpages/Admin.Master" AutoEventWireup="true" CodeBehind="Login.aspx.cs" Inherits="RJLou.Admin.Login1" %>

<asp:Content ContentPlaceHolderID="MainContent" runat="server">
    <div class="section">
        <div class="container">
            <asp:Panel runat="server" DefaultButton="submitButton">
            <h1>Please Log In</h1>
            <asp:Label runat="server" ID="ErrorText" Text="Unfortunately, your login attempt has failed." Visible="false" /><br />
            
            Email Address:<br /> 
            <asp:TextBox runat="server" ID="Email" /><br />
            
            Password: <br />
            <asp:TextBox runat="server" ID="Password" TextMode="Password" /><br />
            
            <asp:LinkButton runat="server" ID="submitButton" CssClass="button" Text="Log In" OnClick="verifyInfo" />
                </asp:Panel>
        </div>
    </div>
</asp:Content>
{% endhighlight %}

[Admin/Login.aspx.cs](https://github.com/jblackett/RJLou/blob/master/Admin/Login.aspx.cs)
{% highlight c# %}
namespace RJLou.Admin
{
    public partial class Login1 : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            if (Session["PersonID"] != null)
                Response.Redirect("Default.aspx");

            if (Page.IsPostBack)
            {
                ErrorText.Visible = true;
            }
        }

        protected void verifyInfo(object sender, EventArgs e)
        {
            string email = Email.Text;
            string password = Password.Text;

            if (string.IsNullOrEmpty(email) || string.IsNullOrEmpty(password))
                return;

            using (SqlConnection conn = new SqlConnection(Constants.DSN))
            {
                conn.Open();

                string sql = @"
                    SELECT      p.Person_ID
                    FROM        Internal_User iu
                    INNER JOIN  Person p ON iu.Person_ID = p.Person_ID
                    WHERE       p.Email = @Email
                    AND         iu.Password = @Password";

                SqlCommand cmd = new SqlCommand(sql, conn);
                cmd.CommandType = CommandType.Text;
                cmd.Parameters.AddWithValue("Email", email);
                cmd.Parameters.AddWithValue("Password", password);

                SqlDataReader read = cmd.ExecuteReader();

                if (read.Read())
                {
                    Session["PersonID"] = read["Person_ID"].ToString();
                    Response.Redirect("Default.aspx");
                }
            }
        }
    }
}
{% endhighlight %}

[Admin/Default.aspx]()
{% highlight aspx-cs %}
<asp:Content ContentPlaceHolderID="MainContent" runat="server">
    <asp:ScriptManager runat='server' ID='workingMan' />
    <div id="container_left" class="container left">
        <div style="margin: 1rem;">
            <input type="search" class="search" name="searchTxt" placeholder="Start searching..." />
            <a href="#" class="button search">Go</a>
            <div style="display: none;"></div>
        </div>
        <table class="changes" cellspacing="0" border="0">
            <tr>
                <td id="CaseSwitchTDAll" runat="server"><asp:LinkButton runat="server" ID="CaseSwitchAll" OnClick="SwitchCaseList" Text="All" CommandArgument="all" /></td>
                <td id="CaseSwitchTDOpen" runat="server"><asp:LinkButton runat="server" ID="CaseSwitchOpen" OnClick="SwitchCaseList" Text="Open" CommandArgument="open" /></td>
                <td id="CaseSwitchTDPending" runat="server"><asp:LinkButton runat="server" ID="CaseSwitchPending" OnClick="SwitchCaseList" Text="Pending Approval" CommandArgument="pending" /></td>
                <td id="CaseSwitchTDClosed" runat="server"><asp:LinkButton runat="server" ID="CaseSwitchClosed" OnClick="SwitchCaseList" Text="Closed" CommandArgument="closed" /></td>
            </tr>
        </table>
        <asp:Repeater runat="server" ID="CasesRepeater" OnItemDataBound="CasesRepeater_Databind">
            <HeaderTemplate>
                <table cellspacing="0" border="0">
                    <thead>
                        <tr>
                            <th>Case ID</th>
                            <th>Name</th>
                            <th>Status</th>
                        </tr>
                    </thead>
                    <tbody>
            </HeaderTemplate>
            <ItemTemplate>
                <tr>
                    <td><asp:LinkButton runat="server" ID="CaseButton" OnClick="LoadCase" Text='<%# Eval("CaseID") %>' CommandArgument='<%# Eval("CaseID") %>' /></td>
                    <td><asp:Label ID="Name" runat="server"></asp:Label></td>
                    <td><%#DataBinder.Eval(Container.DataItem, "Status")%></td>
                </tr>
            </ItemTemplate>
            <FooterTemplate>
                    </tbody>
                </table>
            </FooterTemplate>
        </asp:Repeater>
    </div>
    <div class="container right" id="RightContainer" runat="server">
        <asp:Panel ID="MainContainer" runat="server" Visible="false">
            <div class="scroll-stick">
                <a class="smaller" href="#case_info">Case Info</a>
                <a class="smaller" href="#victims">Victims</a>
                <a class="smaller" href="#offenders">Offenders</a>
                <a class="smaller" href="#affiliates">Affiliates</a>
                <a class="smaller" href="#notes">Notes</a>
                <a class="smaller" href="#charges">Charges</a>
                <a class="smaller" href="#documents">Documents</a>
                <a class="smaller" href="#casemanagers">Employees</a>
            </div>
            <div style="clear: both;"></div>
            <asp:Panel ID="CaseUpdatedPanel" runat="server" CssClass="updatepanel">
                <p>
                    This case was successfully saved!
                </p>
                <span class="x alert">X</span>
            </asp:Panel>
            <h1 id="case_info" class="statusheader" runat="server"></h1>
            <asp:LinkButton ID="AddNewCase" CssClass="button add" runat="server" Text="+" OnClick="AddCase" />
            <div id="statusbar" class="status">
                <h1 id="Status" runat="server"></h1>
                <div class="status dropdown">
                    <asp:LinkButton ID="StatusOpen" runat="server" Text="Open" OnClick="SetStatus" CommandArgument="Open" />
                    <asp:LinkButton ID="StatusPending" runat="server" Text="Pending Approval" OnClick="SetStatus" CommandArgument="Pending" />
                    <asp:LinkButton ID="StatusClosedSuccess" runat="server" Text="Closed" OnClick="SetStatus" CommandArgument="Closed" />
                    <%--<asp:LinkButton ID="StatusClosedFail" runat="server" Text="Closed Unsuccessfully" OnClick="SetStatus" CommandArgument="ClosedFail" />--%>
                </div>
            </div>
            <table class="nothing main">
                <tr>
                    <td>Case ID:</td>
                    <td><asp:TextBox ID="CaseID" runat="server" /></td>
                </tr>
                <tr>
                    <td>Court ID:</td>
                    <td><asp:TextBox ID="CourtID" runat="server" /></td>
                </tr>
                <tr>
                    <td>District:</td>
                    <td><asp:TextBox ID="District" runat="server" /></td>
                </tr>
                <tr>
                    <td>Referral Date:</td>
                    <td><asp:TextBox ID="ReferralDate" runat="server" /></td>
                </tr>
                <tr>
                    <td>Referral Number:</td>
                    <td><asp:TextBox ID="ReferralNumber" runat="server" /></td>
                </tr>
                <tr>
                    <td>Court Date:</td>
                    <td><asp:TextBox ID="CourtDate" runat="server" /></td>
                </tr>
                <tr>
                    <td>Date of Final Conference:</td>
                    <td><asp:TextBox ID="DateFinalConference" runat="server" /></td>
                </tr>
                <tr>
                    <td>Date of Completion:</td>
                    <td><asp:TextBox ID="DateCompletion" runat="server" /></td>
                </tr>
                <%--<tr>
                    <td>Status:</td>
                    <td><asp:TextBox ID="Status" runat="server" /></td>
                </tr>--%>
            </table>
            <asp:LinkButton ID="LinkButton1" runat="server" CssClass="button" Text="Save Case" OnClick="SaveCase" />
            <%--<asp:LinkButton runat="server" CssClass="button" Text="Save Case" OnClick="SaveCase" />--%>
            <h1 id="victims">Victims</h1>
            <div class="inner">
                <asp:Repeater ID="VictimsRepeater" runat="server" OnItemDataBound="VictimsRepeater_ItemDataBound">
                    <HeaderTemplate>
                        <table cellspacing="0" border="0">
                            <thead>
                                <tr>
                                    <th>Name</th>
                                    <th>Gender</th>
                                    <th>Race</th>
                                    <th>Actions</th>
                                </tr>
                            </thead>
                            <tbody>
                    </HeaderTemplate>
                    <ItemTemplate>
                        <tr>
                            <td><asp:Label ID="VictimName" runat="server" /></td>
                            <td><%#DataBinder.Eval(Container.DataItem, "Gender")%></td>
                            <td><%#DataBinder.Eval(Container.DataItem, "Race")%></td>
                            <td>
                                <asp:LinkButton runat="server" ID="VictimDeleteButton" OnClick="DeleteVictim" Text="Delete" CommandArgument='<%# Eval("PersonID") %>' /> &nbsp;
                                <asp:LinkButton runat="server" ID="VictimViewButton" OnClick="ViewVictim" Text="View" CommandArgument='<%# Eval("PersonID") %>' />
                            </td>
                        </tr>
                    </ItemTemplate>
                    <FooterTemplate>
                            </tbody>
                        </table>
                        <asp:LinkButton runat="server" ID="VictimAddButton" OnClick="AddVictim" CssClass="button float-right" Text="Add Victim" />
                    </FooterTemplate>
                </asp:Repeater>
            </div>
...
{% endhighlight %}

[Admin/Default.aspx.cs]()
{% highlight c# %}
namespace RJLou
{
    public partial class Default : System.Web.UI.Page
    {
        Case thisCase;
        List<Case> cases;
        int PersonID = -1;
        List<Victim> allVictims;
        List<Offender> allOffenders;
        List<Affiliate> allAffiliates;
        List<InternalUser> allEmployees;

        protected void Page_Load(object sender, EventArgs e)
        {
            try { PersonID = Convert.ToInt32(Session["PersonID"]); }
            catch { }

            if (PersonID <= 0)
            {
                Response.Redirect("Login.aspx");
            }

            if (!Page.IsPostBack)
            {
                InternalUser thisUser = InternalUser.Get(PersonID);
                if (thisUser.Role == Role.CASE_MANAGER)
                {
                    cases = Case.GetCases(true, thisUser.PersonID);
                    CasesRepeater.DataSource = cases;
                    CasesRepeater.DataBind();

                    CaseSwitchTDAll.Attributes.Add("class", "active");
                    CaseSwitchTDOpen.Attributes.Remove("class");
                    CaseSwitchTDPending.Attributes.Remove("class");
                    CaseSwitchTDClosed.Attributes.Remove("class");
                }
                else
                {
                    cases = Case.GetCases(true);
                    CasesRepeater.DataSource = cases;
                    CasesRepeater.DataBind();

                    CaseSwitchTDAll.Attributes.Add("class", "active");
                    CaseSwitchTDOpen.Attributes.Remove("class");
                    CaseSwitchTDPending.Attributes.Remove("class");
                    CaseSwitchTDClosed.Attributes.Remove("class");
                }

                if (Request.QueryString["CaseID"] != null)
                {
                    int CaseID = Convert.ToInt32(Request.QueryString["CaseID"]);

                    thisCase = Case.Get(CaseID);

                    Session["CaseID"] = CaseID;

                    BindData();
                    LoadHeader();
                    RightContainer.Attributes.CssStyle["display"] = "block";
                    UnloadCaseButton.CssClass = "undo show";
                }
            }
        }

        protected internal void BindData()
        {
            int caseID = -1;

            try { caseID = thisCase.CaseID; }
            catch { }

            if (caseID <= 0)
                thisCase = Case.Get(Convert.ToInt32(Session["CaseID"]));

            VictimsRepeater.DataSource = thisCase.Victims;
            VictimsRepeater.DataBind();

            OffendersRepeater.DataSource = thisCase.Offenders;
            OffendersRepeater.DataBind();

            AffiliatesRepeater.DataSource = thisCase.Affiliates;
            AffiliatesRepeater.DataBind();

            NotesRepeater.DataSource = thisCase.Notes;
            NotesRepeater.DataBind();

            ChargesRepeater.DataSource = thisCase.Charges;
            ChargesRepeater.DataBind();

            DocumentsRepeater.DataSource = thisCase.Documents;
            DocumentsRepeater.DataBind();

            CaseManagersRepeater.DataSource = thisCase.CaseManagers;
            CaseManagersRepeater.DataBind();
        }

        protected internal void CasesRepeater_Databind(object sender, RepeaterItemEventArgs e)
        {
            if (e.Item.ItemType == ListItemType.Item || e.Item.ItemType == ListItemType.AlternatingItem)
            {
                Label thisLabel = (Label)e.Item.FindControl("Name");
                Case currentCase = (Case)e.Item.DataItem;

                string Name;

                if (currentCase.Offenders.Count > 0)
                    Name = currentCase.Offenders[0].FirstName + " " + currentCase.Offenders[0].LastName;
                else if (currentCase.Victims.Count > 0)
                    Name = currentCase.Victims[0].FirstName + " " + currentCase.Victims[0].LastName;
                else
                    Name = "";

                thisLabel.Text = Name;
            }
        }

        protected void AddCase(object sender, EventArgs e)
        {
            NewCaseModalPanel.CssClass += " visible";
        }

        protected internal void CreateNewCase(object sender, EventArgs e)
        {
            if (ModalCourtID.Text.Length == 0)
                return;
            if (ModalStatus.Text.Length == 0)
                return;

            int newCaseID = Case.Add(
                Convert.ToInt32(ModalCourtID.Text), 
                (ModalReferralDate.Text.Length > 0 ? Convert.ToDateTime(ModalReferralDate.Text) : default(DateTime)),
                (ModalReferralNumber.Text.Length > 0 ? Convert.ToInt32(ModalReferralNumber.Text) : 0), 
                (ModalCourtDate.Text.Length > 0 ? Convert.ToDateTime(ModalCourtDate.Text) : default(DateTime)),
                ModalDistrict.Text, 
                (ModalDateFinalConf.Text.Length > 0 ? Convert.ToDateTime(ModalDateFinalConf.Text) : default(DateTime)),
                (ModalDateCompletion.Text.Length > 0 ? Convert.ToDateTime(ModalDateCompletion.Text) : default(DateTime)), 
                ModalStatus.Text);

            thisCase = Case.Get(newCaseID);
            SwitchCaseList(null, null);
            LoadHeader();
            BindData();

            Session["CaseID"] = CaseID;

            NewCaseModalPanel.CssClass = "modal-background";
        }

        protected internal void CancelNewCase(object sender, EventArgs e)
        {
            NewCaseModalPanel.CssClass = "modal-background";
        }

        protected void SetStatus(object sender, EventArgs e)
        {
            int caseID = -1;

            try { caseID = thisCase.CaseID; }
            catch { }

            if (caseID <= 0)
                thisCase = Case.Get(Convert.ToInt32(Session["CaseID"]));

            string newStatus = (((LinkButton)sender).CommandArgument).ToString();
            thisCase.Status = newStatus;
            LoadHeader();
        }
...
{% endhighlight %}