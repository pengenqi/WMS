# WMS
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using DataAccess.Command;
namespace WCSWebClient.AJAX
{
    /// <summary>
    /// StackOperation 的摘要说明
    /// </summary>
    public class StackOperation : IHttpHandler
    {

        string action = HttpContext.Current.Request.Params["action"];
        string toline = HttpContext.Current.Request.Params["toline"];
        string tocolumn = HttpContext.Current.Request.Params["tocolumn"];
        string tolayer = HttpContext.Current.Request.Params["tolayer"];
        string tostation = HttpContext.Current.Request.Params["tostation"];

        string fromline = HttpContext.Current.Request.Params["fromline"];
        string fromcolumn = HttpContext.Current.Request.Params["fromcolumn"];
        string fromlayer = HttpContext.Current.Request.Params["fromlayer"];
        string fromstation = HttpContext.Current.Request.Params["fromstation"];

        string username = HttpContext.Current.Request.Params["username"];
        string opcCon = "堆垛机";
        public void ProcessRequest(HttpContext context)
        {
            switch (action)
            {
                case "StartingProcess":
                    context.Response.Write(StartingProcess(username));
                    break;
                case "CancelProcess":
                    context.Response.Write(CancelProcess(username));
                    break;
                case "ResetProcess":
                    context.Response.Write(ResetProcess(username));
                    break;
                case "RunningProcess":
                    context.Response.Write(RunningProcess(toline, tocolumn, tolayer, username));
                    break;
                case "PickUpProcess":
                    context.Response.Write(PickUpProcess(fromline, fromcolumn, fromlayer, fromstation, username));
                    break;
                case "ReleaseProcess":
                    context.Response.Write(ReleaseProcess(toline, tocolumn, tolayer, tostation, username));
                    break;
                case "PickUpAndReleaseProcess":
                    context.Response.Write(PickUpAndReleaseProcess(toline, tocolumn, tolayer, tostation, fromline, fromcolumn, fromlayer, fromstation, username));
                    break;
            }
        }

        /// <summary>
        /// 堆垛机联机启动
        /// </summary>
        /// <param name="username"></param>
        /// <returns></returns>
        private string StartingProcess(string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";
                if (VSPEC.WCS.OpcServerOperation.service != null)
                {
                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.Starting.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("堆垛机联机启动成功，操作者：{0}", username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 堆垛机任务取消
        /// </summary>
        /// <param name="username"></param>
        /// <returns></returns>
        private string CancelProcess(string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";
                if (VSPEC.WCS.OpcServerOperation.service != null)
                {
                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.Cancel.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("堆垛机取消任务成功，操作者：{0}", username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 堆垛机故障复位
        /// </summary>
        /// <param name="username"></param>
        /// <returns></returns>
        private string ResetProcess(string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";
                if (VSPEC.WCS.OpcServerOperation.service != null)
                {
                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.Reset.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("堆垛机故障复位成功，操作者：{0}", username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 给堆垛机下发取放货命令，堆垛机将执行取货和放货
        /// </summary>
        /// <param name="toline"></param>
        /// <param name="tocolumn"></param>
        /// <param name="tolayer"></param>
        /// <param name="tostation"></param>
        /// <param name="fromline"></param>
        /// <param name="fromcolumn"></param>
        /// <param name="fromlayer"></param>
        /// <param name="fromstation"></param>
        /// <param name="username"></param>
        /// <returns></returns>
        private string PickUpAndReleaseProcess(string toline, string tocolumn, string tolayer, string tostation, string fromline,
            string fromcolumn, string fromlayer, string fromstation, string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";
                if (VSPEC.WCS.OpcServerOperation.service != null)
                {

                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        if (!string.IsNullOrEmpty(fromline))
                        {
                            var opcitemline = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromLine.ToString());
                            if (null != opcitemline)
                            {
                                opcitemline.WriteValue(fromline);
                            }
                        }

                        if (!string.IsNullOrEmpty(fromcolumn))
                        {
                            var opcitemcolumn = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromColumn.ToString());
                            if (null != opcitemcolumn)
                            {
                                opcitemcolumn.WriteValue(fromcolumn);
                            }
                        }

                        if (!string.IsNullOrEmpty(fromlayer))
                        {
                            var opcitemlayer = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromLayer.ToString());
                            if (null != opcitemlayer)
                            {
                                opcitemlayer.WriteValue(fromlayer);
                            }
                        }

                        if (!string.IsNullOrEmpty(fromstation))
                        {
                            var opcitemstation = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromStation.ToString());
                            if (null != opcitemstation)
                            {
                                opcitemstation.WriteValue(fromstation);
                            }
                        }

                        if (!string.IsNullOrEmpty(toline))
                        {
                            var opcitemline = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLine.ToString());
                            if (null != opcitemline)
                            {
                                opcitemline.WriteValue(toline);
                            }
                        }

                        if (!string.IsNullOrEmpty(tocolumn))
                        {
                            var opcitemcolumn = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToColumn.ToString());
                            if (null != opcitemcolumn)
                            {
                                opcitemcolumn.WriteValue(tocolumn);
                            }
                        }

                        if (!string.IsNullOrEmpty(tolayer))
                        {
                            var opcitemlayer = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLayer.ToString());
                            if (null != opcitemlayer)
                            {
                                opcitemlayer.WriteValue(tolayer);
                            }
                        }

                        if (!string.IsNullOrEmpty(tostation))
                        {
                            var opcitemstation = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToStation.ToString());
                            if (null != opcitemstation)
                            {
                                opcitemstation.WriteValue(tostation);
                            }
                        }


                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.PickupAndRelease.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        var opcitemtasknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackTaskNo.ToString());
                        if (null != opcitemtasknumber)
                        {
                            var tasknumber = int.Parse(svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.R_StackTaskNo.ToString()).ReadValue().ToString()) + 1;
                            opcitemtasknumber.WriteValue(tasknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("下发堆垛机取放货命令成功！出发巷道：{0}，出发列：{1}，出发层：{2}，取货站台：{3}，目的巷道：{4}，目的列：{5}，目的层：{6}，放货站台：{7}， 操作者：{8}",
                           fromline, fromcolumn, fromlayer, fromstation, toline, tocolumn, tolayer, tostation, username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 给堆垛机下发 运行指令，堆垛机运行到指定位置
        /// </summary>
        /// <param name="line"></param>
        /// <param name="column"></param>
        /// <param name="layer"></param>
        /// <param name="userName"></param>
        /// <returns></returns>
        private string RunningProcess(string line, string column, string layer, string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";

                if (VSPEC.WCS.OpcServerOperation.service != null)
                {

                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        if (!string.IsNullOrEmpty(line))
                        {
                            var opcitemline = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLine.ToString());
                            if (null != opcitemline)
                            {
                                opcitemline.WriteValue(line);
                            }
                        }
                        else
                            rel = "请指定巷道！";

                        if (!string.IsNullOrEmpty(column))
                        {
                            var opcitemcolumn = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToColumn.ToString());
                            if (null != opcitemcolumn)
                            {
                                opcitemcolumn.WriteValue(column);
                            }
                        }

                        if (!string.IsNullOrEmpty(layer))
                        {
                            var opcitemlayer = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLayer.ToString());
                            if (null != opcitemlayer)
                            {
                                opcitemlayer.WriteValue(layer);
                            }
                        }

                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.Running.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        var opcitemtasknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackTaskNo.ToString());
                        if (null != opcitemtasknumber)
                        {
                            var tasknumber = int.Parse(svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.R_StackTaskNo.ToString()).ReadValue().ToString()) + 1;
                            opcitemtasknumber.WriteValue(tasknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("下发堆垛机运行命令成功！巷道：{0}，列：{1}，层：{2}，操作者：{3}", line, column, layer, username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 给堆垛机下发放货指令，堆垛机将货放到指定的货位或站台
        /// </summary>
        /// <param name="line"></param>
        /// <param name="column"></param>
        /// <param name="layer"></param>
        /// <param name="station"></param>
        /// <param name="username"></param>
        /// <returns></returns>
        private string ReleaseProcess(string line, string column, string layer, string station, string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";

                if (VSPEC.WCS.OpcServerOperation.service != null)
                {

                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        if (!string.IsNullOrEmpty(line))
                        {
                            var opcitemline = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLine.ToString());
                            if (null != opcitemline)
                            {
                                opcitemline.WriteValue(line);
                            }
                        }

                        if (!string.IsNullOrEmpty(column))
                        {
                            var opcitemcolumn = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToColumn.ToString());
                            if (null != opcitemcolumn)
                            {
                                opcitemcolumn.WriteValue(column);
                            }
                        }

                        if (!string.IsNullOrEmpty(layer))
                        {
                            var opcitemlayer = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToLayer.ToString());
                            if (null != opcitemlayer)
                            {
                                opcitemlayer.WriteValue(layer);
                            }
                        }

                        if (!string.IsNullOrEmpty(station))
                        {
                            var opcitemstation = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackToStation.ToString());
                            if (null != opcitemstation)
                            {
                                opcitemstation.WriteValue(station);
                            }
                        }

                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.Release.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        var opcitemtasknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackTaskNo.ToString());
                        if (null != opcitemtasknumber)
                        {
                            var tasknumber = int.Parse(svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.R_StackTaskNo.ToString()).ReadValue().ToString()) + 1;
                            opcitemtasknumber.WriteValue(tasknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("下发堆垛机放货命令成功！巷道：{0}，列：{1}，层：{2}，放货站台：{3}， 操作者：{4}", line, column, layer, station, username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }

        /// <summary>
        /// 给堆垛机下发取货指令，堆垛机将从指定的货位或站台去取货
        /// </summary>
        /// <param name="line"></param>
        /// <param name="column"></param>
        /// <param name="layer"></param>
        /// <param name="station"></param>
        /// <param name="username"></param>
        /// <returns></returns>
        private string PickUpProcess(string line, string column, string layer, string station, string username)
        {
            string rel = "";
            try
            {
                if (VSPEC.WCS.OpcServerOperation.service == null)
                    return rel = "请确认是否开启服务";
                if (VSPEC.WCS.OpcServerOperation.service != null)
                {

                    var svcDevice = VSPEC.WCS.OpcServerOperation.service.Devices;
                    if (null != svcDevice && null != svcDevice.FirstOrDefault())
                    {
                        if (!string.IsNullOrEmpty(line))
                        {
                            var opcitemline = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromLine.ToString());
                            if (null != opcitemline)
                            {
                                opcitemline.WriteValue(line);
                            }
                        }

                        if (!string.IsNullOrEmpty(column))
                        {
                            var opcitemcolumn = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromColumn.ToString());
                            if (null != opcitemcolumn)
                            {
                                opcitemcolumn.WriteValue(column);
                            }
                        }

                        if (!string.IsNullOrEmpty(layer))
                        {
                            var opcitemlayer = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromLayer.ToString());
                            if (null != opcitemlayer)
                            {
                                opcitemlayer.WriteValue(layer);
                            }
                        }

                        if (!string.IsNullOrEmpty(station))
                        {
                            var opcitemstation = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackFromStation.ToString());
                            if (null != opcitemstation)
                            {
                                opcitemstation.WriteValue(station);
                            }
                        }

                        var opcitemworknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackWorkNumber.ToString());
                        if (null != opcitemworknumber)
                        {
                            var worknumber = (int)Enum.Parse(typeof(StacktaskType), StacktaskType.PickUp.ToString());
                            opcitemworknumber.WriteValue(worknumber);
                        }

                        var opcitemtasknumber = svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.W_StackTaskNo.ToString());
                        if (null != opcitemtasknumber)
                        {
                            var tasknumber = int.Parse(svcDevice.FirstOrDefault(r => r.ItemName == OPCSCDBName.R_StackTaskNo.ToString()).ReadValue().ToString()) + 1;
                            opcitemtasknumber.WriteValue(tasknumber);
                        }

                        DataAccess.Funcation.Insert.WCSDataInsertOperation.InsertTrackInfo(
                            string.Format("下发堆垛机取货命令成功！巷道：{0}，列：{1}，层：{2}，取货站台：{3}， 操作者：{4}", line, column, layer, station, username),
                            false, rel, "");

                    }
                }
            }
            catch (Exception ex)
            {
                rel = ex.Message;
            }
            return rel;
        }


        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
